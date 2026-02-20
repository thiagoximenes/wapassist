# 03 — Backend: Node.js + Fastify

> **Objetivo:** Construir a API REST completa com autenticação JWT, CRUD de clientes, webhook do Mercado Pago, integração com WhatsApp e scheduler de cobranças automáticas.  
> **Stack:** Node.js + Fastify + Prisma ORM  
> **Hospedagem:** Render (free tier)  
> **Tempo estimado:** 4–6 horas de desenvolvimento

---

## Estrutura de Arquivos

```
itaflix-api/
├── src/
│   ├── server.js              # Entry point — registra plugins, rotas e inicia o servidor
│   ├── prisma.js              # Singleton do PrismaClient
│   ├── routes/
│   │   ├── auth.js            # POST /api/auth/login — retorna JWT
│   │   ├── clients.js         # CRUD completo de clientes
│   │   ├── payments.js        # Histórico de pagamentos + webhook Mercado Pago
│   │   ├── notes.js           # CRUD de notas por cliente
│   │   └── events.js          # CRUD de eventos do calendário
│   ├── services/
│   │   ├── whatsapp.js        # Integração com Evolution API
│   │   ├── mercadopago.js     # Geração de link Pix e consulta de pagamento
│   │   ├── billing.js         # Cálculo de datas e preços por plano
│   │   ├── scheduler.js       # node-cron: cobranças D-1, inadimplência, calendário
│   │   └── recurrence.js      # Geração de ocorrências de eventos recorrentes
│   └── middleware/
│       └── auth.js            # Verificação do JWT nas rotas protegidas
├── prisma/
│   └── schema.prisma          # Schema do banco (ver 02-DATABASE.md)
├── .env                       # Variáveis de ambiente (nunca commitar)
├── .env.example               # Template de variáveis (commitar sem valores)
└── package.json
```

---

## Etapa 3.1 — Criar o Projeto e Instalar Dependências

```bash
mkdir itaflix-api && cd itaflix-api
npm init -y
npm install fastify @fastify/cors @fastify/jwt dotenv
npm install @prisma/client mercadopago node-cron axios date-fns
npm install -D prisma nodemon
npx prisma init --datasource-provider postgresql
```

Adicionar ao `package.json`:

```json
{
  "type": "module",
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js"
  }
}
```

### Checklist

- [ ] Projeto criado com `npm init -y`
- [ ] Todas as dependências instaladas sem erros
- [ ] `"type": "module"` adicionado ao `package.json`
- [ ] Prisma inicializado: pasta `prisma/` criada com `schema.prisma`

---

## Etapa 3.2 — Entry Point (`src/server.js`)

```javascript
import Fastify from 'fastify';
import cors from '@fastify/cors';
import jwt from '@fastify/jwt';
import 'dotenv/config';

import { authRoutes }     from './routes/auth.js';
import { clientRoutes }   from './routes/clients.js';
import { paymentRoutes }  from './routes/payments.js';
import { noteRoutes }     from './routes/notes.js';
import { eventRoutes }    from './routes/events.js';
import { startScheduler } from './services/scheduler.js';

const app = Fastify({ logger: true });

await app.register(cors, { origin: process.env.FRONTEND_URL });
await app.register(jwt,  { secret: process.env.JWT_SECRET });

app.get('/health', async () => ({ status: 'ok', timestamp: new Date() }));

await app.register(authRoutes,    { prefix: '/api' });
await app.register(clientRoutes,  { prefix: '/api' });
await app.register(paymentRoutes, { prefix: '/api' });
await app.register(noteRoutes,    { prefix: '/api' });
await app.register(eventRoutes,   { prefix: '/api' });

startScheduler();

const port = Number(process.env.PORT) || 3000;
await app.listen({ port, host: '0.0.0.0' });
console.log(`Servidor Itaflix rodando na porta ${port}`);
```

---

## Etapa 3.3 — Middleware de Autenticação (`src/middleware/auth.js`)

```javascript
export async function authenticate(request, reply) {
  try {
    await request.jwtVerify();
  } catch {
    reply.status(401).send({ error: 'Token inválido ou ausente' });
  }
}
```

---

## Etapa 3.4 — Rota de Autenticação (`src/routes/auth.js`)

```javascript
import { authenticate } from '../middleware/auth.js';

export async function authRoutes(fastify) {
  // POST /api/auth/login
  fastify.post('/auth/login', async (req, reply) => {
    const { password } = req.body;
    if (password !== process.env.ADMIN_PASSWORD) {
      return reply.status(401).send({ error: 'Senha incorreta' });
    }
    const token = fastify.jwt.sign({ role: 'admin' }, { expiresIn: '7d' });
    return { token };
  });

  // GET /api/auth/me — valida token e retorna status
  fastify.get('/auth/me', { preHandler: authenticate }, async () => {
    return { ok: true, role: 'admin' };
  });
}
```

---

## Etapa 3.5 — Serviço de Billing (`src/services/billing.js`)

Este é o núcleo da regra de negócio. **Não alterar sem testes.**

```javascript
export const PLAN_DAYS = {
  MONTHLY:    30,
  QUARTERLY:  90,
  SEMIANNUAL: 180,
  ANNUAL:     365,
};

export const PLAN_PRICES = {
  MONTHLY:    30,
  QUARTERLY:  80,
  SEMIANNUAL: 150,
  ANNUAL:     280,
};

export const PLAN_LABELS = {
  MONTHLY:    'Mensal',
  QUARTERLY:  'Trimestral',
  SEMIANNUAL: 'Semestral',
  ANNUAL:     'Anual',
};

/**
 * Calcula a nova data de vencimento após um pagamento.
 * Regra:
 *   - Se pagou ANTES ou NO DIA do vencimento: nova = dueDate + dias do plano
 *   - Se pagou DEPOIS do vencimento: nova = paidAt + dias do plano
 */
export function calculateNewDueDate(currentDueDate, paidAt, plan) {
  const due  = new Date(currentDueDate);
  const paid = new Date(paidAt);
  const days = PLAN_DAYS[plan];

  const base = paid <= due ? due : paid;
  const next = new Date(base);
  next.setDate(next.getDate() + days);
  return next;
}

export function getPlanPrice(plan) {
  return PLAN_PRICES[plan] ?? 0;
}

export function getPlanLabel(plan) {
  return PLAN_LABELS[plan] ?? plan;
}
```

---

## Etapa 3.6 — Serviço WhatsApp (`src/services/whatsapp.js`)

```javascript
import axios from 'axios';

const BASE_URL  = process.env.EVOLUTION_URL;
const API_KEY   = process.env.EVOLUTION_APIKEY;
const INSTANCE  = process.env.EVOLUTION_INSTANCE;
const ADMIN     = process.env.ADMIN_PHONE;

async function sendMessage(phone, text) {
  try {
    await axios.post(
      `${BASE_URL}/message/sendText/${INSTANCE}`,
      { number: phone, text },
      { headers: { apikey: API_KEY } }
    );
  } catch (err) {
    console.error(`[WhatsApp] Erro ao enviar para ${phone}:`, err.message);
    // Não lança exceção — não pode derrubar o fluxo principal
  }
}

export async function sendBillingReminder(phone, name, dueDate, pixLink) {
  const data = new Date(dueDate).toLocaleDateString('pt-BR');
  await sendMessage(phone,
    `Olá, *${name}*! 👋\n` +
    `Sua assinatura *Itaflix* vence amanhã, *${data}*.\n\n` +
    `Para renovar, pague o Pix abaixo:\n` +
    `🔗 ${pixLink}\n\n` +
    `O link expira em 48 horas. Qualquer dúvida é só chamar! 😊`
  );
}

export async function sendPaymentConfirmation(phone, name, newDueDate, plan) {
  const data  = new Date(newDueDate).toLocaleDateString('pt-BR');
  const label = getPlanLabel(plan);
  await sendMessage(phone,
    `✅ *Pagamento confirmado!*\n\n` +
    `Olá, *${name}*!\n` +
    `Recebemos seu pagamento com sucesso.\n\n` +
    `📅 Nova validade: *${data}*\n` +
    `📦 Plano: ${label}\n\n` +
    `Bom entretenimento! 🎬\n— Equipe Itaflix`
  );
}

export async function sendAdminAlert(message) {
  await sendMessage(ADMIN, `⚠️ *Alerta Itaflix*\n\n${message}`);
}
```

---

## Etapa 3.7 — Serviço Mercado Pago (`src/services/mercadopago.js`)

```javascript
import MercadoPagoConfig, { Preference, Payment } from 'mercadopago';

const mpClient = new MercadoPagoConfig({
  accessToken: process.env.MP_ACCESS_TOKEN,
});

export async function createPixLink({ title, amount, externalReference }) {
  const preference = new Preference(mpClient);
  const result = await preference.create({
    body: {
      items: [{ title, quantity: 1, unit_price: amount }],
      payment_methods: {
        excluded_payment_types: [
          { id: 'credit_card' },
          { id: 'debit_card' },
        ],
      },
      external_reference: externalReference, // phone do cliente
      expiration_date_to: new Date(Date.now() + 48 * 60 * 60 * 1000).toISOString(),
    },
  });
  return result.init_point; // URL do checkout Pix
}

export async function getPayment(paymentId) {
  const payment = new Payment(mpClient);
  return payment.get({ id: paymentId });
}
```

---

## Etapa 3.8 — Rotas de Clientes (`src/routes/clients.js`)

### Endpoints

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `GET` | `/api/clients` | JWT | Lista todos os clientes com filtros opcionais |
| `GET` | `/api/clients/:id` | JWT | Detalhe do cliente com pagamentos e notas |
| `POST` | `/api/clients` | JWT | Cadastrar novo cliente |
| `PUT` | `/api/clients/:id` | JWT | Atualizar dados do cliente |
| `DELETE` | `/api/clients/:id` | JWT | Soft delete (muda status para INACTIVE) |
| `POST` | `/api/clients/:id/send-billing` | JWT | Enviar cobrança manual via WhatsApp |

### Query params para `GET /api/clients`

| Param | Tipo | Descrição |
|---|---|---|
| `status` | string | Filtrar por `ACTIVE`, `OVERDUE` ou `INACTIVE` |
| `plan` | string | Filtrar por plano |
| `search` | string | Busca por nome ou telefone |

### Campo calculado `daysUntilDue`

Retornado em cada cliente. Pode ser negativo (cliente em atraso):

```javascript
const daysUntilDue = Math.ceil(
  (new Date(client.dueDate) - new Date()) / (1000 * 60 * 60 * 24)
);
```

### Formato do telefone

Ao salvar, remover caracteres não numéricos e garantir formato `55DDNÚMERO`:

```javascript
const phone = body.phone.replace(/\D/g, '');
const formatted = phone.startsWith('55') ? phone : `55${phone}`;
```

### Checklist

- [ ] `GET /api/clients` retorna lista ordenada por `dueDate` asc
- [ ] `GET /api/clients/:id` inclui `payments` e `notes`
- [ ] `POST /api/clients` valida telefone único e plano válido
- [ ] `DELETE /api/clients/:id` faz soft delete (status INACTIVE)
- [ ] `POST /api/clients/:id/send-billing` gera Pix e envia WhatsApp

---

## Etapa 3.9 — Webhook Mercado Pago (`src/routes/payments.js`)

> ⚠️ Esta rota **não tem autenticação JWT** — é chamada diretamente pelo Mercado Pago.

### Fluxo do webhook

```
MP chama POST /api/webhook/mercadopago
    │
    ├── type !== 'payment' → retorna 200 (ignorar)
    │
    ├── Busca pagamento na API do MP pelo data.id
    │
    ├── status !== 'approved' → retorna 200 (ignorar)
    │
    ├── Busca cliente pelo external_reference (phone)
    │
    ├── Não encontrou → sendAdminAlert + retorna 200
    │
    ├── Calcula nova data de vencimento (billing.calculateNewDueDate)
    │
    ├── Salva Payment no banco
    │
    ├── Atualiza Client: dueDate + status = ACTIVE
    │
    └── sendPaymentConfirmation → retorna { ok: true }
```

### Checklist

- [ ] Webhook registrado no painel do Mercado Pago
- [ ] Rota processa apenas pagamentos `approved`
- [ ] Cliente identificado pelo `external_reference` (phone)
- [ ] Nova data calculada corretamente pela regra de negócio
- [ ] Confirmação enviada via WhatsApp após pagamento

---

## Etapa 3.10 — Scheduler (`src/services/scheduler.js`)

### 4 Jobs Automáticos

| Job | Horário (cron) | Função |
|---|---|---|
| **Job 1** — Cobrança D-1 | `0 9 * * *` (09h diário) | Envia cobrança para clientes que vencem amanhã |
| **Job 2** — Inadimplência | `0 10 * * *` (10h diário) | Marca clientes como OVERDUE, envia aviso WhatsApp ao cliente e cria notificação na dashboard para o admin |
| **Job 3** — Notificações calendário | `*/5 * * * *` (a cada 5 min) | Envia notificações de eventos do calendário |
| **Job 4** — Expandir recorrências | `0 7 * * 1` (07h toda segunda) | Garante 90 dias de eventos recorrentes no banco |

### Job 1 — Cobrança D-1

```javascript
// Roda todo dia às 09:00 (horário de Brasília)
cron.schedule('0 9 * * *', async () => {
  const tomorrow = new Date();
  tomorrow.setDate(tomorrow.getDate() + 1);

  const clients = await prisma.client.findMany({
    where: {
      dueDate: { gte: startOfDay(tomorrow), lte: endOfDay(tomorrow) },
      status: 'ACTIVE',
    },
  });

  for (const client of clients) {
    try {
      const pixLink = await createPixLink({
        title: `Itaflix - Renovação ${getPlanLabel(client.plan)}`,
        amount: getPlanPrice(client.plan),
        externalReference: client.phone,
      });
      await sendBillingReminder(client.phone, client.name, tomorrow, pixLink);
      console.log(`Cobrança enviada para ${client.name} - vence em ${tomorrow.toLocaleDateString('pt-BR')}`);
    } catch (err) {
      console.error(`Erro ao cobrar ${client.name}:`, err.message);
    }
  }
});
```

### Job 2 — Inadimplência (D+3)

```javascript
// Roda todo dia às 10:00
cron.schedule('0 10 * * *', async () => {
  const threeDaysAgo = new Date();
  threeDaysAgo.setDate(threeDaysAgo.getDate() - 3);

  const overdue = await prisma.client.findMany({
    where: {
      dueDate: { lt: threeDaysAgo },
      status: { not: 'INACTIVE' },
    },
  });

  if (overdue.length === 0) return;

  // Atualiza status para OVERDUE no banco
  await prisma.client.updateMany({
    where: { id: { in: overdue.map(c => c.id) } },
    data: { status: 'OVERDUE' },
  });

  // Envia aviso de inadimplência via WhatsApp para cada cliente
  for (const client of overdue) {
    try {
      const dias = Math.floor((new Date() - new Date(client.dueDate)) / 86400000);
      await sendOverdueNotice(client.phone, client.name, dias);
      console.log(`Aviso de inadimplência enviado para ${client.name} (${dias} dias em atraso)`);
    } catch (err) {
      console.error(`Erro ao notificar ${client.name}:`, err.message);
    }
  }

  // Registra alerta no banco para exibição na dashboard (sem WhatsApp para o admin)
  // A dashboard consome esse dado via GET /api/dashboard/summary → campo 'alerts'
  console.log(`Job inadimplência: ${overdue.length} cliente(s) marcado(s) como OVERDUE`);
});
```

### Checklist

- [ ] Job 1 envia cobranças às 09h para clientes com vencimento amanhã
- [ ] Job 2 marca clientes como OVERDUE após 3 dias, envia aviso WhatsApp ao cliente e registra alerta na dashboard
- [ ] Job 3 envia notificações de calendário a cada 5 minutos
- [ ] Job 4 expande recorrências toda segunda às 07h
- [ ] Logs descritivos em português para cada job

---

## Etapa 3.11 — Rota de Notas (`src/routes/notes.js`)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `POST` | `/api/notes` | JWT | Criar nota para um cliente |
| `DELETE` | `/api/notes/:id` | JWT | Deletar nota |

Body para `POST /api/notes`:
```json
{ "clientId": 1, "content": "Cliente pediu para mudar para plano trimestral" }
```

---

## Etapa 3.12 — Endpoint de Dashboard Summary

Endpoint auxiliar para a tela de Visão Geral do frontend:

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `GET` | `/api/dashboard/summary` | JWT | KPIs consolidados |

Retorno esperado:
```json
{
  "activeClients": 28,
  "dueSoon": 5,
  "overdue": 3,
  "monthRevenue": 840.00,
  "todayBillings": [...],
  "recentPayments": [...],
  "alerts": [...]
}
```

---

## Etapa 3.13 — Endpoint de Status do WhatsApp

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `GET` | `/api/whatsapp/status` | JWT | Verifica se a instância está conectada |

```javascript
fastify.get('/whatsapp/status', { preHandler: authenticate }, async () => {
  try {
    const res = await axios.get(
      `${process.env.EVOLUTION_URL}/instance/fetchInstances`,
      { headers: { apikey: process.env.EVOLUTION_APIKEY } }
    );
    const instance = res.data.find(i => i.instance.instanceName === process.env.EVOLUTION_INSTANCE);
    return { connected: instance?.instance?.state === 'open' };
  } catch {
    return { connected: false };
  }
});
```

---

## Checklist Geral do Backend

- [ ] Projeto criado e dependências instaladas
- [ ] Schema Prisma aplicado no banco (`npx prisma db push`)
- [ ] `.env` preenchido com todas as variáveis
- [ ] `GET /health` retorna `{ status: 'ok' }`
- [ ] `POST /api/auth/login` retorna JWT válido
- [ ] CRUD de clientes funcionando (criar, listar, editar, inativar)
- [ ] Webhook do Mercado Pago processando pagamentos
- [ ] Scheduler iniciando com os 4 jobs
- [ ] WhatsApp enviando mensagens de teste
- [ ] Backend deployado no Render com URL pública
