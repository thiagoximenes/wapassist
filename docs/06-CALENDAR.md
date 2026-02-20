# 06 — Funcionalidade: Calendário

> **Objetivo:** Implementar a tela de calendário que unifica cobranças automáticas de clientes, tarefas pessoais únicas e tarefas recorrentes, com notificações via WhatsApp.  
> **Escopo:** Fase 1 do MVP (pode ser implementado após o core estar estável)  
> **Tempo estimado:** 4–6 horas

---

## O que é o Calendário?

O calendário unifica **três origens diferentes de eventos** em uma única visualização:

| Cor | Tipo | Origem | Exemplos |
|---|---|---|---|
| 🔵 Azul | **Cobrança** | Gerado automaticamente pelo sistema | Vencimento de assinatura de cliente |
| 🟣 Roxo | **Tarefa única** | Criada manualmente ou via WhatsApp | Reunião dia 12/06 às 19h |
| 🟡 Amarelo | **Tarefa recorrente** | Criada com regra de repetição | Remédio toda sexta às 12h |
| 🔴 Vermelho | **Atrasado** | Qualquer evento não concluído no prazo | — |

---

## Etapa 6.1 — Schema do Banco (já incluído em `02-DATABASE.md`)

As tabelas `CalendarEvent` e `Recurrence` já estão no schema Prisma. Referência rápida:

### Exemplos de eventos mapeados para o schema

| Exemplo | `category` | `freq` | `weekDay` / `monthDay` | `notifyAt` |
|---|---|---|---|---|
| Reunião dia 12/06/2026 às 19h | `REMINDER` | `ONCE` | — | 1h antes |
| Pagar imposto em 12/04/2026 às 13h | `TASK` | `ONCE` | — | 1 dia antes |
| Remédio toda sexta-feira às 12h | `RECURRING` | `WEEKLY` | `weekDay: 5` | no horário |
| Pagar internet todo dia 15 — R$100 | `RECURRING` | `MONTHLY_DAY` | `monthDay: 15` | 1 dia antes |
| Cobrança João Silva (mensal) | `BILLING` | `ONCE` | — | 1 dia antes |

> ℹ️ Cobranças são criadas como `ONCE` porque cada renovação gera um novo evento com nova data. O sistema as cria automaticamente ao processar o webhook de pagamento.

---

## Etapa 6.2 — Dependências do Frontend

```bash
# Calendário visual (leve, suporta mês e semana nativamente)
npm install @fullcalendar/react @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction

# Date picker para o modal de criação
npm install react-datepicker
```

---

## Etapa 6.3 — Rotas da API de Eventos (`src/routes/events.js`)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `GET` | `/api/events` | JWT | Lista eventos com filtros: `?month=3&year=2026` ou `?week=2026-03-09` |
| `GET` | `/api/events/:id` | JWT | Detalhe de um evento específico |
| `POST` | `/api/events` | JWT | Criar evento único (`TASK` ou `REMINDER`) |
| `POST` | `/api/events/recurring` | JWT | Criar evento recorrente + gera próximas ocorrências |
| `PUT` | `/api/events/:id` | JWT | Editar evento (título, hora, `notifyAt`, `amount`) |
| `PATCH` | `/api/events/:id/done` | JWT | Marcar evento como concluído |
| `DELETE` | `/api/events/:id` | JWT | Apaga evento único ou uma ocorrência |
| `DELETE` | `/api/events/:id/all` | JWT | Apaga evento e todas as ocorrências futuras da recorrência |
| `POST` | `/api/events/:id/notify` | JWT | Disparar notificação WhatsApp manualmente |

---

## Etapa 6.4 — Serviço de Recorrências (`src/services/recurrence.js`)

```javascript
import { addDays, addWeeks, addMonths, addYears } from 'date-fns';

/**
 * Gera as datas de ocorrência de uma recorrência
 * a partir de hoje até 'horizonDays' dias à frente.
 */
export function generateOccurrences(recurrence, horizonDays = 90) {
  const { freq, weekDay, monthDay, startDate, endDate } = recurrence;
  const occurrences = [];
  const horizon     = addDays(new Date(), horizonDays);
  let   cursor      = new Date(Math.max(new Date(startDate), new Date()));

  while (cursor <= horizon && (!endDate || cursor <= new Date(endDate))) {
    occurrences.push(new Date(cursor));
    switch (freq) {
      case 'DAILY':       cursor = addDays(cursor, 1);   break;
      case 'WEEKLY':      cursor = addWeeks(cursor, 1);  break;
      case 'MONTHLY_DAY': cursor = addMonths(cursor, 1); break;
      case 'YEARLY':      cursor = addYears(cursor, 1);  break;
      default: return occurrences; // ONCE — apenas uma ocorrência
    }
  }
  return occurrences;
}

/**
 * Cria os CalendarEvents no banco para cada ocorrência.
 * Evita duplicatas verificando startAt + recurrenceId já existentes.
 */
export async function materializeRecurrence(prisma, recurrence, eventTemplate) {
  const dates   = generateOccurrences(recurrence);
  const created = [];

  for (const date of dates) {
    const exists = await prisma.calendarEvent.findFirst({
      where: { recurrenceId: recurrence.id, startAt: date },
    });
    if (exists) continue;

    const notifyAt = eventTemplate.notifyAt
      ? addDays(date, -1)  // padrão: notificar 1 dia antes
      : date;              // ou no horário do evento

    created.push(await prisma.calendarEvent.create({
      data: { ...eventTemplate, startAt: date, notifyAt, recurrenceId: recurrence.id },
    }));
  }
  return created;
}
```

---

## Etapa 6.5 — Jobs do Scheduler para o Calendário

Adicionar ao `src/services/scheduler.js` existente:

### Job 3 — Notificações de Eventos (a cada 5 minutos)

```javascript
import { addMinutes } from 'date-fns';

cron.schedule('*/5 * * * *', async () => {
  const now  = new Date();
  const soon = addMinutes(now, 5);

  const pending = await prisma.calendarEvent.findMany({
    where: {
      notifyWhatsApp: true,
      notified:       false,
      done:           false,
      notifyAt:       { gte: now, lte: soon },
    },
    include: { client: true },
  });

  for (const event of pending) {
    try {
      await sendEventNotification(event);
      await prisma.calendarEvent.update({
        where: { id: event.id },
        data:  { notified: true },
      });
      console.log(`Notificação enviada: [${event.category}] ${event.title}`);
    } catch (err) {
      console.error(`Erro ao notificar evento ${event.id}:`, err.message);
    }
  }
});
```

### Job 4 — Expandir Recorrências (toda segunda às 07h)

```javascript
cron.schedule('0 7 * * 1', async () => {
  const recurrences = await prisma.recurrence.findMany({
    where: { OR: [{ endDate: null }, { endDate: { gte: new Date() } }] },
  });

  for (const rec of recurrences) {
    const tmpl = await prisma.calendarEvent.findFirst({
      where: { recurrenceId: rec.id },
    });
    if (tmpl) await materializeRecurrence(prisma, rec, tmpl);
  }

  console.log(`Recorrências expandidas: ${recurrences.length} regras processadas`);
});
```

---

## Etapa 6.6 — Mensagens de Notificação por Categoria

Adicionar à função `sendEventNotification` em `src/services/whatsapp.js`:

```javascript
export async function sendEventNotification(event) {
  const date = new Date(event.startAt).toLocaleDateString('pt-BR', {
    day: '2-digit', month: '2-digit', year: 'numeric',
    hour: '2-digit', minute: '2-digit',
  });

  switch (event.category) {
    case 'BILLING':
      // Enviada para o CLIENTE
      await sendMessage(event.client.phone,
        `💳 *Itaflix — Cobrança*\n\n` +
        `Olá, *${event.client.name}*!\n` +
        `Sua assinatura vence amanhã, *${date}*.\n\n` +
        `💰 Valor: R$ ${event.amount}\n` +
        `🔗 ${event.pixLink || 'Em breve'}`
      );
      break;

    case 'TASK':
    case 'REMINDER':
      // Enviada para o ADMIN
      await sendAdminAlert(
        `🔔 *Lembrete*\n\n` +
        `📌 ${event.title}\n` +
        `🕐 ${date}` +
        (event.description ? `\n📝 ${event.description}` : '') +
        (event.amount ? `\n💰 R$ ${event.amount}` : '')
      );
      break;

    case 'RECURRING':
      await sendAdminAlert(
        `🔁 *Tarefa recorrente*\n\n` +
        `📌 ${event.title}\n` +
        `🕐 ${date}` +
        (event.amount ? `\n💰 R$ ${event.amount}` : '')
      );
      break;
  }
}
```

---

## Etapa 6.7 — Tela do Calendário (`src/pages/CalendarPage.jsx`)

### View Mensal

```
┌─ TOPBAR ──────────────────────────────────────────────────────────────────┐
│  Calendário              [← Fev]  Março 2026  [Abr →]   [Mês] [Semana]  │
│                                                          [+ Nova Tarefa]  │
├────────────────────────────────────────────────────────────────────────────┤
│  🔵 Cobrança   🟣 Tarefa única   🟡 Recorrente   🔴 Atrasado  ← Legenda  │
│                                                                             │
│  Dom    Seg    Ter    Qua    Qui    Sex    Sáb                             │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                       │
│  │    │ │    │ │  3 │ │  4 │ │  5 │ │  6 │ │  7 │                       │
│  │    │ │    │ │    │ │🔵·2│ │    │ │🟡  │ │    │                       │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘                       │
│  ...                                                                        │
│  → Clicar em dia: abre painel lateral com eventos do dia                  │
│  → Clicar em evento: abre modal de detalhe/edição                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

### View Semanal (grade por hora)

```
┌─ TOPBAR ──────────────────────────────────── [← Semana]  09–15 Mar  [→] ─┐
│       Dom 9  Seg 10  Ter 11  Qua 12  Qui 13  Sex 14  Sáb 15              │
│  08h                                                                        │
│  09h           ┌──────────────┐                                             │
│                │ 🔵 Cobrança  │                                             │
│  10h           │ Ana Costa    │                                             │
│                │ R$ 30 · Mens.│                                             │
│  11h           └──────────────┘                                             │
│  12h                                    ┌────────┐  ┌────────────────┐    │
│                                         │🟣Reunião│  │🟡 Remédio     │    │
│  13h                                    │19h     │  │ toda sexta 12h│    │
│                                         └────────┘  └────────────────┘    │
└────────────────────────────────────────────────────────────────────────────┘
```

### Painel Lateral — Eventos do Dia

```
┌─ Eventos — Sexta, 14 de Março ──────────────────┐
│                                  [+ Adicionar]   │
│  ─── COBRANÇAS ────────────────────────────────  │
│  🔵 09:00  Cobrança — João Silva                │
│            Mensal · R$30 · [✓ Enviada] [Ver →]  │
│                                                   │
│  ─── TAREFAS ──────────────────────────────────  │
│  🟡 12:00  Remédio (recorrente · toda sexta)    │
│            [✓ Marcar feito]                      │
│                                                   │
│  ─── LEMBRETES ────────────────────────────────  │
│  🟣 19:00  Reunião com fornecedor                │
│            [✓ Marcar feito]  [Editar]  [🗑]     │
└──────────────────────────────────────────────────┘
```

### Modal de Criação/Edição de Evento

**Campos:**
- Título (obrigatório)
- Tipo: radio — Tarefa única / Tarefa recorrente / Lembrete
- Data e hora (`datetime-local`)
- Valor em R$ (opcional)
- Descrição (textarea, opcional)
- Toggle "Notificar no WhatsApp" (default: ligado)
- Antecedência: No horário / 30min antes / 1h antes / 1 dia antes

**Se tipo = Tarefa recorrente, mostrar painel extra:**
- Frequência: Diária / Semanal / Mensal (dia do mês) / Anual
- Se Semanal: checkboxes de dias da semana
- Se Mensal: input "Todo dia ___ do mês"
- Data de término (opcional)

---

## Etapa 6.8 — CSS do FullCalendar (`src/styles/calendar.css`)

```css
/* Fundo geral */
.fc { --fc-border-color: var(--border); }
.fc-theme-standard td,
.fc-theme-standard th { border-color: var(--border); }
.fc-scrollgrid { background: var(--bg-surface); }

/* Cabeçalho dos dias */
.fc-col-header-cell { background: var(--bg-panel); }
.fc-col-header-cell-cushion {
  color: var(--text-secondary);
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  text-decoration: none;
}

/* Células dos dias */
.fc-daygrid-day { background: var(--bg-surface); }
.fc-daygrid-day:hover { background: var(--bg-hover); }
.fc-daygrid-day-number {
  color: var(--text-secondary);
  font-size: 0.8rem;
  text-decoration: none;
}

/* Dia atual */
.fc-day-today { background: var(--bg-active) !important; }
.fc-day-today .fc-daygrid-day-number { color: var(--cyan-400); }

/* Eventos concluídos */
.fc-event-done { opacity: 0.4; }

/* Remover toolbar padrão (usamos nosso próprio header) */
.fc-header-toolbar { display: none; }
```

---

## Mapeamento de Cores dos Eventos para o FullCalendar

```javascript
const CATEGORY_COLORS = {
  BILLING:   { backgroundColor: '#06B6D4', borderColor: '#0891B2' },
  TASK:      { backgroundColor: '#8B5CF6', borderColor: '#7C3AED' },
  RECURRING: { backgroundColor: '#F59E0B', borderColor: '#D97706' },
  REMINDER:  { backgroundColor: '#EC4899', borderColor: '#DB2777' },
};

// Transformar eventos da API para o formato do FullCalendar
const fcEvents = events.map(event => ({
  id:    String(event.id),
  title: event.title,
  start: event.startAt,
  ...CATEGORY_COLORS[event.category],
  classNames: event.done ? ['fc-event-done'] : [],
  extendedProps: event,
}));
```

---

## Checklist do Calendário

- [ ] Tabelas `CalendarEvent` e `Recurrence` criadas no banco
- [ ] Rotas da API de eventos implementadas (`src/routes/events.js`)
- [ ] Serviço de recorrências implementado (`src/services/recurrence.js`)
- [ ] Job 3 (notificações a cada 5 min) adicionado ao scheduler
- [ ] Job 4 (expandir recorrências toda segunda) adicionado ao scheduler
- [ ] `sendEventNotification` implementado no `whatsapp.js`
- [ ] Dependências do frontend instaladas (FullCalendar + react-datepicker)
- [ ] Tela do calendário com view mensal e semanal
- [ ] Modal de criação de evento (único e recorrente)
- [ ] Painel lateral de eventos do dia
- [ ] Modal de detalhe/edição de evento
- [ ] Rota `/calendario` adicionada ao `App.jsx` e à sidebar
