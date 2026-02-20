# 07 — Fase 8: Camada de IA (WhatsApp Inteligente)

> **Status:** � Implementar após Fase 7 (Calendário) estar estável  
> **Posição no roadmap:** Fase 8 de 10 — ver `AGENT-DIRECTOR.md`  
> **Motivo do posicionamento:** Todos os serviços (WhatsApp, billing, scheduler, clientes, notas, calendário) precisam existir e estar testados antes da IA poder orquestrá-los  
> **Custo adicional:** ~US$ 2–5/mês (OpenAI API com volume de uso pessoal)

---

## O que será possível fazer

Com o MVP funcionando, a camada de IA permitirá interagir com o sistema diretamente pelo WhatsApp, como se estivesse conversando com um assistente pessoal:

| Você diz / escreve | O sistema faz |
|---|---|
| "Anota que o João pediu para mudar para plano trimestral" | Salva uma nota na ficha do cliente João no banco |
| "Quais clientes vencem essa semana?" | Consulta o banco e responde com a lista formatada |
| "Me lembra amanhã às 10h de ligar para a Maria" | Cria um lembrete; no horário você recebe a notificação |
| "João pagou, pode confirmar" | Registra o pagamento manualmente, calcula nova data e envia confirmação ao João |
| Mensagem de voz com qualquer comando acima | Transcreve com Whisper (OpenAI) e executa a mesma lógica |

---

## Arquitetura da Camada de IA

```
[Você envia mensagem de texto ou voz no WhatsApp]
    │
    ▼
[Evolution API recebe e chama webhook POST /api/webhook/whatsapp]
    │
    ├── Se for áudio → Whisper API (OpenAI) transcreve para texto
    │
    ▼
[GPT-4o Mini analisa a intenção e extrai entidades]
    │
    ├── Retorna JSON: { intent, clientName?, phone?, content?, date?, time? }
    │
    ▼
[Backend executa a ação correspondente]
    │
    ├── ADD_NOTE       → salva nota no banco
    ├── LIST_CLIENTS   → consulta banco, formata resposta
    ├── REGISTER_PAYMENT → registra pagamento manual
    ├── CREATE_REMINDER → salva agendamento no banco
    ├── LIST_OVERDUE   → lista clientes em atraso
    └── UNKNOWN        → responde com ajuda
    │
    ▼
[Resultado enviado de volta via WhatsApp para você]
```

### Componentes por Tecnologia

| Componente | Tecnologia | Custo estimado |
|---|---|---|
| Entrada de texto | Webhook da Evolution API | R$ 0 |
| Entrada de voz | Whisper API (OpenAI) | ~US$ 0,006/min |
| Interpretação de intenção | GPT-4o Mini (OpenAI) | ~US$ 0,15/1M tokens |
| Execução | Backend Fastify existente | R$ 0 |
| Resposta | Evolution API (WhatsApp) | R$ 0 |

---

## Etapa 7.1 — Novas Variáveis de Ambiente

Adicionar ao `.env` do backend quando implementar:

```env
# OpenAI
OPENAI_API_KEY=sk-...

# Número do WhatsApp do admin (só aceita comandos deste número)
MY_WHATSAPP=5521999998888
```

---

## Etapa 7.2 — Webhook de Mensagens Recebidas

**Arquivo:** `src/routes/whatsapp.js` (novo arquivo)

```javascript
// POST /api/webhook/whatsapp (sem JWT — chamado pela Evolution API)
fastify.post('/webhook/whatsapp', async (req, reply) => {
  const { sender, messageType, content } = extractMessage(req.body);

  // Segurança: só aceita comandos do número do admin
  if (sender !== process.env.MY_WHATSAPP) {
    return reply.send({ ok: true }); // ignora silenciosamente
  }

  let text = content;

  // Se for áudio, transcreve com Whisper
  if (messageType === 'audio') {
    text = await transcribeAudio(content); // baixa o áudio e envia para Whisper
  }

  // Processa o comando com IA
  await processCommand(text, sender);

  return reply.send({ ok: true });
});
```

### Configurar o Webhook na Evolution API

```bash
curl -X POST "https://api.wapassist.com.br/webhook/set/wapassist" \
  -H "Content-Type: application/json" \
  -H "apikey: SUA_CHAVE" \
  -d '{
    "url": "https://wapassist-api.onrender.com/api/webhook/whatsapp",
    "webhook_by_events": false,
    "events": ["MESSAGES_UPSERT"]
  }'
```

---

## Etapa 7.3 — Transcrição de Áudio com Whisper

**Arquivo:** `src/services/whisper.js`

```javascript
import OpenAI from 'openai';
import axios from 'axios';
import fs from 'fs';
import path from 'path';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export async function transcribeAudio(audioUrl) {
  // Baixa o arquivo de áudio temporariamente
  const response = await axios.get(audioUrl, { responseType: 'arraybuffer' });
  const tmpPath  = path.join('/tmp', `audio_${Date.now()}.ogg`);
  fs.writeFileSync(tmpPath, response.data);

  try {
    const transcription = await openai.audio.transcriptions.create({
      file:  fs.createReadStream(tmpPath),
      model: 'whisper-1',
      language: 'pt',
    });
    return transcription.text;
  } finally {
    fs.unlinkSync(tmpPath); // limpa o arquivo temporário
  }
}
```

---

## Etapa 7.4 — Assistente de IA (`src/services/aiAssistant.js`)

```javascript
import OpenAI from 'openai';
import { prisma }      from '../prisma.js';
import { sendAdminAlert, sendPaymentConfirmation } from './whatsapp.js';
import { calculateNewDueDate, getPlanLabel }        from './billing.js';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const SYSTEM_PROMPT = `
Você é o assistente do wapassist, sistema de gestão de assinaturas IPTV.
Analise a mensagem do usuário e retorne um JSON com a intenção identificada.

Intenções possíveis:
- ADD_NOTE: adicionar anotação sobre um cliente
- LIST_CLIENTS: listar clientes (com filtro opcional: vencimento, status)
- REGISTER_PAYMENT: registrar pagamento manual de um cliente
- CREATE_REMINDER: criar lembrete pessoal com data/hora
- LIST_OVERDUE: listar clientes em atraso
- UNKNOWN: intenção não reconhecida

Retorne APENAS o JSON, sem texto adicional:
{
  "intent": "ADD_NOTE",
  "clientName": "João Silva",
  "phone": null,
  "content": "Pediu para mudar para plano trimestral",
  "date": null,
  "time": null
}
`;

export async function processCommand(userMessage, senderPhone) {
  // 1. Interpreta a intenção com GPT-4o Mini
  const completion = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [
      { role: 'system', content: SYSTEM_PROMPT },
      { role: 'user',   content: userMessage },
    ],
    response_format: { type: 'json_object' },
  });

  const { intent, clientName, content, date, time } =
    JSON.parse(completion.choices[0].message.content);

  // 2. Executa a ação correspondente
  switch (intent) {
    case 'ADD_NOTE': {
      const client = await findClientByName(clientName);
      if (!client) {
        await sendAdminAlert(`Não encontrei cliente com o nome "${clientName}".`);
        return;
      }
      await prisma.note.create({ data: { clientId: client.id, content } });
      await sendAdminAlert(`✅ Nota adicionada para *${client.name}*:\n"${content}"`);
      break;
    }

    case 'LIST_CLIENTS': {
      const clients = await prisma.client.findMany({
        where: { status: 'ACTIVE' },
        orderBy: { dueDate: 'asc' },
        take: 10,
      });
      const lista = clients.map(c => {
        const dias = Math.ceil((new Date(c.dueDate) - new Date()) / 86400000);
        return `• ${c.name} — vence em ${dias} dias`;
      }).join('\n');
      await sendAdminAlert(`📋 *Próximos vencimentos:*\n\n${lista}`);
      break;
    }

    case 'REGISTER_PAYMENT': {
      const client = await findClientByName(clientName);
      if (!client) {
        await sendAdminAlert(`Não encontrei cliente com o nome "${clientName}".`);
        return;
      }
      const newDue = calculateNewDueDate(client.dueDate, new Date(), client.plan);
      await prisma.payment.create({
        data: { clientId: client.id, paidAt: new Date(), newDueDate: newDue, amount: 0 },
      });
      await prisma.client.update({
        where: { id: client.id },
        data: { dueDate: newDue, status: 'ACTIVE' },
      });
      await sendPaymentConfirmation(client.phone, client.name, newDue, client.plan);
      await sendAdminAlert(`✅ Pagamento de *${client.name}* registrado. Nova validade: ${newDue.toLocaleDateString('pt-BR')}`);
      break;
    }

    case 'CREATE_REMINDER': {
      const scheduledAt = parseDateTime(date, time); // usa date-fns para parsing
      await prisma.calendarEvent.create({
        data: {
          title: content || userMessage,
          category: 'REMINDER',
          startAt: scheduledAt,
          notifyAt: scheduledAt,
          notifyWhatsApp: true,
        },
      });
      await sendAdminAlert(`⏰ Lembrete criado: "${content}" para ${scheduledAt.toLocaleString('pt-BR')}`);
      break;
    }

    case 'LIST_OVERDUE': {
      const overdue = await prisma.client.findMany({
        where: { status: 'OVERDUE' },
        orderBy: { dueDate: 'asc' },
      });
      if (overdue.length === 0) {
        await sendAdminAlert('✅ Nenhum cliente em atraso no momento!');
        return;
      }
      const lista = overdue.map(c => {
        const dias = Math.floor((new Date() - new Date(c.dueDate)) / 86400000);
        return `• ${c.name} — ${dias} dias em atraso`;
      }).join('\n');
      await sendAdminAlert(`⚠️ *Clientes em atraso (${overdue.length}):*\n\n${lista}`);
      break;
    }

    default:
      await sendAdminAlert(
        `Não entendi o comando. Tente:\n` +
        `• "Anota que [cliente] [observação]"\n` +
        `• "Listar clientes"\n` +
        `• "Clientes em atraso"\n` +
        `• "[cliente] pagou"\n` +
        `• "Me lembra [data] às [hora] de [tarefa]"`
      );
  }
}

async function findClientByName(name) {
  return prisma.client.findFirst({
    where: { name: { contains: name, mode: 'insensitive' } },
  });
}
```

---

## Etapa 7.5 — Sistema de Lembretes Pessoais

Os lembretes criados via IA já são salvos como `CalendarEvent` com `category: REMINDER`. O Job 3 do scheduler (a cada 5 minutos) já os processa automaticamente.

Para suporte a parsing de datas em português ("amanhã às 10h", "sexta às 15h30"):

```javascript
import { parseISO, addDays, setHours, setMinutes, nextDay } from 'date-fns';
import { ptBR } from 'date-fns/locale';

function parseDateTime(dateStr, timeStr) {
  const now = new Date();

  // Parsing básico de datas relativas em português
  if (dateStr?.toLowerCase() === 'amanhã') {
    const base = addDays(now, 1);
    if (timeStr) return applyTime(base, timeStr);
    return base;
  }

  // Para datas absolutas, usar parseISO ou date-fns/parse
  // Implementar conforme necessidade
  return now;
}

function applyTime(date, timeStr) {
  const [h, m] = timeStr.replace('h', ':').split(':').map(Number);
  return setMinutes(setHours(date, h), m || 0);
}
```

---

## Checklist da Fase IA

> ⚠️ Implementar somente após o MVP estar estável em produção.

- [ ] Conta OpenAI criada e `OPENAI_API_KEY` obtida
- [ ] `MY_WHATSAPP` configurado no `.env`
- [ ] Webhook de mensagens recebidas configurado na Evolution API
- [ ] Rota `POST /api/webhook/whatsapp` implementada
- [ ] Serviço Whisper para transcrição de áudio implementado
- [ ] `aiAssistant.js` com todos os intents implementado
- [ ] Parsing de datas em português funcionando
- [ ] Teste: enviar "listar clientes" via WhatsApp e receber resposta
- [ ] Teste: enviar áudio com comando e receber resposta
- [ ] Teste: criar lembrete via WhatsApp e receber notificação no horário
