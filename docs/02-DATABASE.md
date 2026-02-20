# 02 — Banco de Dados: PostgreSQL no Neon.tech

> **Objetivo:** Criar e configurar o banco de dados PostgreSQL gerenciado, definir o schema completo via Prisma e garantir que as tabelas estejam prontas para o backend.  
> **Tecnologia:** PostgreSQL hospedado no [Neon.tech](https://neon.tech) (free tier) + Prisma ORM  
> **Tempo estimado:** 30–60 minutos

---

## Por que Neon.tech?

- PostgreSQL gerenciado, sem necessidade de configurar na VPS
- Free tier: 0,5 GB de armazenamento (suficiente para centenas de clientes)
- Região disponível: AWS São Paulo (`sa-east-1`) — baixa latência
- Suspende após 7 dias sem atividade (o ping do Render/UptimeRobot evita isso)

---

## Etapa 2.1 — Criar a Conta e o Banco

1. Acesse [neon.tech](https://neon.tech) e crie uma conta gratuita (login com Google funciona)
2. Clique em **New Project** e nomeie como `wapassist`
3. Selecione a região: **AWS São Paulo (sa-east-1)**
4. Após criar, clique em **Connection Details** e copie a **Connection String**

O formato da string é:
```
postgresql://usuario:senha@host.neon.tech/wapassist?sslmode=require
```

> ⚠️ Salve essa string em local seguro. Ela será usada como `DATABASE_URL` no `.env` do backend.

### Checklist

- [ ] Conta Neon.tech criada
- [ ] Projeto `wapassist` criado na região `sa-east-1`
- [ ] Connection String copiada e salva com segurança
- [ ] `DATABASE_URL` preenchida no `.env` do backend

---

## Etapa 2.2 — Schema Prisma Completo

O Prisma ORM é responsável por criar e gerenciar as tabelas. O schema abaixo é o **schema completo do MVP + Calendário**.

**Arquivo:** `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─── Enums ────────────────────────────────────────────────────────────────────

enum Plan {
  MONTHLY     // 30 dias — R$ 30
  QUARTERLY   // 90 dias — R$ 80
  SEMIANNUAL  // 180 dias — R$ 150
  ANNUAL      // 365 dias — R$ 280
}

enum ClientStatus {
  ACTIVE    // em dia
  OVERDUE   // em atraso (passou do vencimento)
  INACTIVE  // desativado manualmente
}

enum RecurrenceFreq {
  DAILY        // todo dia
  WEEKLY       // toda semana (usa weekDay)
  MONTHLY_DAY  // todo dia X do mês (usa monthDay)
  YEARLY       // todo ano nessa data
  ONCE         // evento único (sem recorrência)
}

enum EventCategory {
  BILLING    // cobrança de cliente — gerado pelo sistema
  TASK       // tarefa pessoal única
  RECURRING  // tarefa recorrente
  REMINDER   // lembrete avulso (agendado via WhatsApp/IA)
}

// ─── Models MVP ───────────────────────────────────────────────────────────────

model Client {
  id        Int          @id @default(autoincrement())
  name      String       @db.VarChar(100)
  phone     String       @unique @db.VarChar(20)  // formato: 5521999998888
  email     String?      @db.VarChar(100)
  plan      Plan
  status    ClientStatus @default(ACTIVE)
  dueDate   DateTime     @db.Date                 // próximo vencimento
  createdAt DateTime     @default(now())
  updatedAt DateTime     @updatedAt

  payments       Payment[]
  notes          Note[]
  calendarEvents CalendarEvent[]
}

model Payment {
  id           Int      @id @default(autoincrement())
  clientId     Int
  mpPaymentId  String?  @db.VarChar(100)  // ID do Mercado Pago
  amount       Decimal  @db.Decimal(10, 2)
  paidAt       DateTime
  newDueDate   DateTime @db.Date
  createdAt    DateTime @default(now())

  client Client @relation(fields: [clientId], references: [id])
}

model Note {
  id        Int      @id @default(autoincrement())
  clientId  Int
  content   String
  createdAt DateTime @default(now())

  client Client @relation(fields: [clientId], references: [id])
}

// ─── Models Calendário ────────────────────────────────────────────────────────

model CalendarEvent {
  id             Int           @id @default(autoincrement())
  title          String
  description    String?
  category       EventCategory
  startAt        DateTime
  amount         Decimal?      @db.Decimal(10, 2)
  notifyWhatsApp Boolean       @default(true)
  notifyAt       DateTime?     // quando enviar a notificação (null = no startAt)
  notified       Boolean       @default(false)
  done           Boolean       @default(false)
  clientId       Int?
  recurrenceId   Int?
  createdAt      DateTime      @default(now())

  client     Client?     @relation(fields: [clientId], references: [id])
  recurrence Recurrence? @relation(fields: [recurrenceId], references: [id])
}

model Recurrence {
  id        Int            @id @default(autoincrement())
  freq      RecurrenceFreq
  weekDay   Int?           // 0=Dom, 1=Seg ... 6=Sáb (para WEEKLY)
  monthDay  Int?           // 1-31 (para MONTHLY_DAY)
  startDate DateTime
  endDate   DateTime?      // null = sem fim
  createdAt DateTime       @default(now())

  events CalendarEvent[]
}
```

---

## Etapa 2.3 — Aplicar o Schema no Banco

Após preencher `DATABASE_URL` no `.env`:

```bash
# Aplica o schema no banco (cria/atualiza tabelas)
npx prisma db push

# Gera o Prisma Client (necessário após qualquer mudança no schema)
npx prisma generate

# Abre o Prisma Studio para visualizar os dados (opcional)
npx prisma studio
```

> ✅ O Prisma Studio abre em `localhost:5555` e mostra todas as tabelas visualmente.

### Checklist

- [ ] `npx prisma db push` executado sem erros
- [ ] `npx prisma generate` executado
- [ ] Tabelas `Client`, `Payment`, `Note`, `CalendarEvent`, `Recurrence` existem no banco
- [ ] Prisma Studio confirma as tabelas (opcional)

---

## Etapa 2.4 — Singleton do Prisma Client

Crie o arquivo de conexão singleton para evitar múltiplas instâncias em desenvolvimento:

**Arquivo:** `src/prisma.js`

```javascript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis;

export const prisma =
  globalForPrisma.prisma ?? new PrismaClient({ log: ['error'] });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

---

## Estrutura das Tabelas — Referência Rápida

### `Client` — Clientes

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Int (PK) | ID auto-incremento |
| `name` | String | Nome completo |
| `phone` | String (unique) | Telefone no formato `5521999998888` |
| `email` | String? | E-mail (opcional) |
| `plan` | Enum Plan | `MONTHLY` / `QUARTERLY` / `SEMIANNUAL` / `ANNUAL` |
| `status` | Enum ClientStatus | `ACTIVE` / `OVERDUE` / `INACTIVE` |
| `dueDate` | Date | Próxima data de vencimento |
| `createdAt` | DateTime | Data de cadastro |

### `Payment` — Pagamentos

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Int (PK) | ID auto-incremento |
| `clientId` | Int (FK) | Referência ao cliente |
| `mpPaymentId` | String? | ID do pagamento no Mercado Pago |
| `amount` | Decimal | Valor pago |
| `paidAt` | DateTime | Data/hora do pagamento |
| `newDueDate` | Date | Nova data de vencimento calculada |

### `Note` — Notas por Cliente

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Int (PK) | ID auto-incremento |
| `clientId` | Int (FK) | Referência ao cliente |
| `content` | String | Texto da nota |
| `createdAt` | DateTime | Data da nota |

### `CalendarEvent` — Eventos do Calendário

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Int (PK) | ID auto-incremento |
| `title` | String | Título do evento |
| `category` | Enum EventCategory | `BILLING` / `TASK` / `RECURRING` / `REMINDER` |
| `startAt` | DateTime | Data/hora do evento |
| `amount` | Decimal? | Valor (opcional) |
| `notifyWhatsApp` | Boolean | Se deve notificar via WhatsApp |
| `notifyAt` | DateTime? | Quando enviar a notificação |
| `notified` | Boolean | Se já foi notificado |
| `done` | Boolean | Se foi marcado como concluído |
| `clientId` | Int? (FK) | Referência ao cliente (para cobranças) |
| `recurrenceId` | Int? (FK) | Referência à regra de recorrência |

### `Recurrence` — Regras de Recorrência

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | Int (PK) | ID auto-incremento |
| `freq` | Enum RecurrenceFreq | `DAILY` / `WEEKLY` / `MONTHLY_DAY` / `YEARLY` / `ONCE` |
| `weekDay` | Int? | Dia da semana (0=Dom … 6=Sáb) — para `WEEKLY` |
| `monthDay` | Int? | Dia do mês (1–31) — para `MONTHLY_DAY` |
| `startDate` | DateTime | Data de início da recorrência |
| `endDate` | DateTime? | Data de fim (null = sem fim) |

---

## Observações Importantes

- **Fuso horário:** Sempre use UTC no backend. Converta para `America/Sao_Paulo` apenas na exibição no frontend.
- **Campo `dueDate`:** Salvo como `DATE` (sem hora). Nunca salvar com hora para evitar bugs de comparação.
- **Neon.tech free tier:** Suspende após 7 dias sem atividade. O UptimeRobot pingando o backend a cada 5 min evita isso automaticamente.
- **Migrations vs `db push`:** Em desenvolvimento use `npx prisma db push`. Em produção, considere usar `prisma migrate deploy` para rastrear mudanças.
