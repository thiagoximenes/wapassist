# 09 — Checklist Mestre de Implementação

> **Como usar:** Marque cada item com `[x]` conforme for concluindo. Siga a ordem das fases — cada fase depende da anterior estar completa.  
> **Última atualização:** 21/02/2026

---

## Fase 0 — Pré-requisitos e Contas ✅

- [x] **VPS contratada** — Hostinger KVM 1, Ubuntu 22.04 LTS
- [x] **Conta GitHub** — 3 repositórios criados: `wapassist`, `wapassist-api`, `wapassist-dashboard`
- [x] **Conta Neon.tech** — projeto `neondb`, região `sa-east-1`
- [x] **Conta Render** — conta criada
- [x] **Conta Vercel** — conta criada + MCP vinculado
- [x] **Conta Mercado Pago** — Access Token de produção ativo
- [x] **Conta UptimeRobot** — conta criada
- [x] **Número WhatsApp dedicado** — usando número `5522997309370` (mesmo do admin)

---

## Fase 1 — Infraestrutura (VPS + WhatsApp) ✅

### 1.1 — VPS
- [x] Acesso SSH funcionando
- [x] UFW ativo (portas 22, 80, 443)
- [x] Docker instalado e rodando
- [x] Nginx instalado e rodando

### 1.2 — DNS (Cloudflare — yootiq.com)
- [x] `apiwapassist.yootiq.com` → A → `72.61.57.129` (propagado)
- [x] `adminwapassist.yootiq.com` → CNAME → Vercel ✅

### 1.3 — Evolution API
- [x] Container `evolution_api` rodando (v1.8.7)
- [x] Nginx como proxy reverso para porta 8080
- [x] SSL ativo em `https://apiwapassist.yootiq.com`

### 1.4 — WhatsApp
- [x] Instância `wapassist` criada
- [x] QR Code escaneado — conectada
- [x] Mensagem de teste enviada com sucesso

---

## Fase 2 — Banco de Dados ✅

- [x] Neon.tech — connection string configurada
- [x] Schema Prisma criado em `prisma/schema.prisma`
- [x] `npx prisma db push` executado sem erros
- [x] Tabelas criadas: `Client`, `Payment`, `Note`, `CalendarEvent`, `Recurrence`, `ClientLog`
- [x] Singleton `src/prisma.js` criado

---

## Fase 3 — Backend ✅

### 3.1 — Setup
- [x] Projeto Node.js + Fastify v5 + Prisma v5 (ESM)
- [x] Todas as dependências instaladas
- [x] `.env` preenchido (exceto `MP_WEBHOOK_SECRET`)
- [x] `.env.example` criado

### 3.2 — Serviços Core
- [x] `src/prisma.js` — singleton PrismaClient
- [x] `src/middleware/auth.js` — verificação JWT
- [x] `src/services/billing.js` — `calculateNewDueDate` com 31 dias/mês fixo
- [x] `src/services/whatsapp.js` — 5 funções: billing, confirmation, overdue, adminAlert, eventNotification
- [x] `src/services/mercadopago.js` — `createPixLink`, `getPayment`

### 3.3 — Rotas
- [x] `src/routes/auth.js` — login + me
- [x] `src/routes/clients.js` — CRUD + send-billing + send-confirmation + logs
- [x] `src/routes/payments.js` — histórico + webhook MP
- [x] `src/routes/notes.js` — criar e deletar
- [x] `src/routes/events.js` — CRUD de eventos
- [x] `src/routes/dashboard.js` — summary KPIs
- [x] `src/routes/whatsapp.js` — status + qrcode
- [x] `src/server.js` — entry point com todas as rotas

### 3.4 — Scheduler
- [x] Job 1: cobrança D-1 às 09h
- [x] Job 2: inadimplência às 10h
- [x] Job 3: notificações de eventos a cada 5 min
- [x] Job 4: expandir recorrências toda segunda

### 3.5 — Extras implementados além do plano original
- [x] `POST /api/clients/:id/send-confirmation` — registra pagamento, recalcula vencimento, envia WhatsApp
- [x] `GET /api/logs` — todos os ClientLogs com filtro e include de cliente
- [x] `GET /api/clients/:id/logs` — logs por cliente
- [x] `GET /api/whatsapp/qrcode` — retorna QR Code para reconexão
- [x] Preço customizado por cliente (`price` field no Client)

---

## Fase 4 — Frontend ✅

### 4.1 — Setup
- [x] Vite + React 19 + TailwindCSS 3
- [x] react-router-dom v6, @tanstack/react-query, axios, date-fns, lucide-react
- [x] CSS variables design system (dark theme)
- [x] Fontes DM Sans + DM Mono
- [x] `src/lib/api.js` com interceptors JWT + 401 redirect
- [x] `src/lib/queryClient.js`
- [x] `src/App.jsx` com roteamento + PrivateRoute
- [x] `.env.local` com `VITE_API_URL=http://localhost:3000`

### 4.2 — Componentes UI
- [x] `Badge` — status e planos (inline-flex, fit-content)
- [x] `Button` — primary, ghost, danger + loading spinner
- [x] `Card` — container padrão
- [x] `Input` — com label, ícone e estado de erro
- [x] `Select` — dropdown estilizado
- [x] `Modal` — overlay com blur
- [x] `Toast` — notificações temporárias com contexto
- [x] `Skeleton` — placeholder de loading
- [x] `Avatar` — inicial com cor gerada (seguro contra undefined/null)
- [x] `StatCard` — card de KPI
- [x] `EmptyState` — tela vazia com ação
- [x] `Tooltip` — tooltip simples

### 4.3 — Layout
- [x] `Layout.jsx` — sidebar + topbar + WhatsApp status + QR code modal + settings dropdown

### 4.4 — Páginas
- [x] `/login` — autenticação JWT
- [x] `/` — KPIs, agenda do dia, alertas, atividade recente
- [x] `/clientes` — tabela com filtros, paginação, menu de ações (badges centralizados, z-index dropdown)
- [x] `/clientes/novo` — formulário com datetime-local para vencimento, preço customizado
- [x] `/clientes/:id` — ficha completa, botões icon-only com tooltip e cores (amarelo/verde), modal de confirmação antes de enviar
- [x] `/clientes/:id/editar` — pré-preenchido com datetime-local
- [x] `/pagamentos` — histórico com KPIs, filtros de período, p.client.name/plan corretos, colunas centralizadas
- [x] `/calendario` — calendário de eventos
- [x] `/templates` — templates editáveis com localStorage, restore-to-default
- [x] `/logs` — logs reais do DB, compacto, filtros por tipo, auto-refresh 30s

### 4.5 — Hooks
- [x] `useClientFilters.js` — filtros e paginação

---

## Fase 5 — Integrações ⏳

- [x] `EVOLUTION_APIKEY`, `EVOLUTION_URL`, `EVOLUTION_INSTANCE` configurados
- [x] `ADMIN_PHONE` configurado
- [x] MP_ACCESS_TOKEN configurado
- [x] **Webhook MP cadastrado no painel** — `https://wapassist-api.onrender.com/api/webhook/mercadopago`
- [x] **`MP_WEBHOOK_SECRET`** — configurado no Render
- [x] Mensagem de cobrança testada em produção
- [ ] Confirmação testada via webhook real (PIX end-to-end pendente)

---

## Fase 6 — Deploy ⏳

- [x] `wapassist-api` no GitHub (branch `main`) ✅
- [x] `wapassist-dashboard` no GitHub (branch `main`) ✅
- [x] **Backend deployado no Render** — `https://wapassist-api.onrender.com` ✅
- [x] `GET https://wapassist-api.onrender.com/health` retorna `{ status: 'ok' }` ✅
- [x] **Frontend deployado na Vercel** — `https://adminwapassist.yootiq.com` ✅
- [x] Dashboard acessível pela URL da Vercel ✅
- [x] **UptimeRobot** configurado pingando `/health` a cada 5 min ✅
- [x] Webhook MP atualizado para URL do Render ✅
- [x] `DIRECT_URL` configurado no Render (sem pooler, para migrations Prisma) ✅
- [ ] `FRONTEND_URL` no Render atualizar para `https://adminwapassist.yootiq.com` ⚠️ pendente

---

## Fase 7 — Validação End-to-End ⬜

- [x] Login na dashboard em produção funcionando ✅
- [x] Cadastrar cliente de teste com telefone real ✅
- [x] Enviar cobrança manual — mensagem WhatsApp recebida ✅
- [ ] Realizar pagamento Pix de teste
- [ ] Webhook processado — log no Render confirma
- [ ] Confirmação de pagamento recebida no WhatsApp
- [ ] Data de vencimento atualizada corretamente
- [ ] Alerta de inadimplência recebido no número admin

---

## Fase 8 — Calendário ✅

- [x] Tabelas `CalendarEvent` e `Recurrence` no schema Prisma
- [x] `npx prisma db push` executado
- [x] Rotas da API de eventos implementadas (`src/routes/events.js`)
- [x] Serviço de recorrências implementado
- [x] Job 3 e Job 4 no scheduler
- [x] `sendEventNotification` no `whatsapp.js`
- [x] Tela `/calendario` implementada
- [x] Rota adicionada ao `App.jsx` e sidebar

---

## Fase 9 — IA (WhatsApp Inteligente) ⬜

> **Pré-requisito:** Conta OpenAI com créditos

- [ ] `OPENAI_API_KEY` e `MY_WHATSAPP` no Render
- [ ] `src/services/whisper.js` — transcrição de áudio
- [ ] `src/services/aiAssistant.js` — intents
- [ ] Webhook de mensagens recebidas na Evolution API
- [ ] Testes de comandos via WhatsApp

---

## Fase 10 — Migração dos Clientes Reais ⬜

- [ ] Listar clientes atuais (nome, telefone, plano, vencimento)
- [ ] Cadastrar na dashboard
- [ ] Verificar datas de vencimento
- [ ] Confirmar scheduler funcionando
- [ ] Sistema em produção com clientes reais ✅

---

## Resumo de Progresso

| Fase | Status | Observações |
|---|---|---|
| Fase 0 — Pré-requisitos | ✅ Concluído | Número WA ainda provisório |
| Fase 1 — Infraestrutura | ✅ Concluído | CNAME Vercel pendente |
| Fase 2 — Banco de Dados | ✅ Concluído | 6 tabelas incluindo ClientLog |
| Fase 3 — Backend Core | ✅ Concluído | branch `main` |
| Fase 4 — Frontend | ✅ Concluído | branch `main`, Vercel auto-deploy ok |
| Fase 5 — Integrações | 🔨 Em andamento | PIX end-to-end pendente |
| Fase 6 — Deploy | ✅ Concluído | Render + Vercel em produção |
| Fase 7 — Validação | 🔨 Em andamento | WhatsApp ok, PIX pendente |
| Fase 8 — Calendário | ✅ Concluído | Backend + frontend |
| Fase 9 — IA | ⬜ Pendente | Requer créditos OpenAI |
| Fase 10 — Migração | ⬜ Pendente | Após validação |
