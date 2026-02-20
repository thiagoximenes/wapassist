# 09 — Checklist Mestre de Implementação

> **Como usar:** Marque cada item com `[x]` conforme for concluindo. Siga a ordem das fases — cada fase depende da anterior estar completa.  
> **Tempo total estimado:** 2–3 semanas (trabalhando em sessões de 2–4 horas)

---

## Fase 0 — Pré-requisitos e Contas

> Tudo que precisa ser criado/contratado antes de escrever uma linha de código.

- [ ] **Domínio `wapassist.com.br`** — acesso ao painel DNS confirmado
- [ ] **VPS contratada** — Ubuntu 22.04 LTS, mínimo 2 GB RAM (Hostinger KVM 1 ou Hetzner CX11)
- [ ] **Conta GitHub** — dois repositórios criados: `wapassist-api` e `wapassist-dashboard`
- [ ] **Conta Neon.tech** — projeto `wapassist` criado na região `sa-east-1`
- [ ] **Conta Render** — conta gratuita criada
- [ ] **Conta Vercel** — conta gratuita criada
- [ ] **Conta Mercado Pago** — conta de vendedor criada, Access Token de produção copiado
- [ ] **Conta UptimeRobot** — conta gratuita criada
- [ ] **Número WhatsApp dedicado** — número separado do pessoal para o wapassist

---

## Fase 1 — Infraestrutura (VPS + WhatsApp)

> Referência: `01-INFRASTRUCTURE.md`

### 1.1 — Configuração da VPS

- [ ] Acesso SSH funcionando: `ssh root@IP_DA_VPS`
- [ ] Sistema atualizado: `apt update && apt upgrade -y`
- [ ] Pacotes base instalados: `curl`, `git`, `ufw`, `nano`
- [ ] Firewall UFW ativo (portas 22, 80, 443 liberadas)
- [ ] Docker instalado: `docker --version` retorna versão
- [ ] Nginx instalado e rodando: `systemctl status nginx` mostra `active`

### 1.2 — DNS

- [ ] Registro DNS tipo A criado: `api.wapassist.com.br → IP_DA_VPS`
- [ ] Propagação confirmada no dnschecker.org

### 1.3 — Evolution API

- [ ] API Key gerada com `openssl rand -hex 32` e salva com segurança
- [ ] Arquivo `~/evolution/docker-compose.yml` criado com a API Key
- [ ] Container `evolution_api` rodando: `docker ps` mostra o container
- [ ] Log confirma: `Server is listening on port 8080`

### 1.4 — Nginx + SSL

- [ ] Nginx configurado como proxy reverso para porta 8080
- [ ] `nginx -t` retorna `syntax is ok`
- [ ] SSL ativo: `https://api.wapassist.com.br` abre sem alertas de segurança

### 1.5 — WhatsApp

- [ ] Instância `wapassist` criada na Evolution API
- [ ] QR Code escaneado com número dedicado
- [ ] Status da instância: `open`
- [ ] Mensagem de teste recebida no WhatsApp

---

## Fase 2 — Banco de Dados

> Referência: `02-DATABASE.md`

- [ ] Projeto `wapassist` criado no Neon.tech (região `sa-east-1`)
- [ ] Connection String copiada e salva com segurança
- [ ] `DATABASE_URL` preenchida no `.env` do backend
- [ ] Schema Prisma completo criado em `prisma/schema.prisma`
- [ ] `npx prisma db push` executado sem erros
- [ ] `npx prisma generate` executado
- [ ] Tabelas `Client`, `Payment`, `Note`, `CalendarEvent`, `Recurrence` existem no banco
- [ ] Singleton `src/prisma.js` criado

---

## Fase 3 — Backend

> Referência: `03-BACKEND.md`

### 3.1 — Setup

- [ ] Projeto `wapassist-api` criado com `npm init -y`
- [ ] Todas as dependências instaladas (Fastify, Prisma, node-cron, axios, mercadopago, etc.)
- [ ] `"type": "module"` adicionado ao `package.json`
- [ ] Estrutura de pastas criada (`src/routes/`, `src/services/`, `src/middleware/`)
- [ ] `.env` preenchido com **todas** as variáveis (nenhuma vazia)
- [ ] `.env.example` criado com placeholders (para commitar no GitHub)

### 3.2 — Serviços Core

- [ ] `src/prisma.js` — singleton do PrismaClient
- [ ] `src/middleware/auth.js` — verificação JWT
- [ ] `src/services/billing.js` — cálculo de datas e preços por plano
- [ ] `src/services/whatsapp.js` — integração Evolution API (3 funções: billing, confirmation, alert)
- [ ] `src/services/mercadopago.js` — geração de link Pix e consulta de pagamento

### 3.3 — Rotas

- [ ] `src/routes/auth.js` — `POST /api/auth/login` retorna JWT válido
- [ ] `src/routes/clients.js` — CRUD completo (GET, POST, PUT, DELETE, send-billing)
- [ ] `src/routes/payments.js` — histórico + webhook Mercado Pago
- [ ] `src/routes/notes.js` — criar e deletar notas
- [ ] `src/server.js` — entry point registrando todas as rotas

### 3.4 — Scheduler

- [ ] `src/services/scheduler.js` — Job 1: cobrança D-1 às 09h
- [ ] `src/services/scheduler.js` — Job 2: inadimplência às 10h
- [ ] Scheduler iniciando com os jobs ao subir o servidor

### 3.5 — Endpoints Auxiliares

- [ ] `GET /health` retorna `{ status: 'ok' }`
- [ ] `GET /api/dashboard/summary` retorna KPIs para a home
- [ ] `GET /api/whatsapp/status` retorna status da instância

### 3.6 — Testes Locais

- [ ] `node src/server.js` sobe sem erros
- [ ] `POST /api/auth/login` com senha correta retorna token
- [ ] `GET /api/clients` retorna lista (pode estar vazia)
- [ ] `POST /api/clients` cria um cliente de teste
- [ ] Webhook MP testado manualmente com `curl`

---

## Fase 4 — Frontend

> Referência: `04-FRONTEND.md`

### 4.1 — Setup e Fundação

- [ ] Projeto criado com Vite + template React
- [ ] Dependências instaladas (axios, react-router-dom, react-query, lucide-react, date-fns)
- [ ] TailwindCSS configurado com `postcss` e `autoprefixer`
- [ ] `src/styles/tokens.css` criado com todas as CSS variables da paleta
- [ ] Fontes DM Mono + DM Sans importadas no `index.html`
- [ ] `src/lib/api.js` com instância axios + interceptors de token e 401
- [ ] `src/lib/queryClient.js` com configuração do React Query
- [ ] `src/App.jsx` com roteamento completo e `PrivateRoute`
- [ ] `.env.local` criado com `VITE_API_URL`

### 4.2 — Componentes UI

- [ ] `ui/Badge.jsx` — status e planos
- [ ] `ui/Button.jsx` — variantes primary, ghost, danger
- [ ] `ui/Card.jsx` — container padrão
- [ ] `ui/Input.jsx` — com label e estado de erro
- [ ] `ui/Select.jsx` — dropdown estilizado
- [ ] `ui/Modal.jsx` — overlay com animação
- [ ] `ui/Toast.jsx` — notificações temporárias
- [ ] `ui/Skeleton.jsx` — placeholder de loading
- [ ] `ui/Avatar.jsx` — inicial do nome com cor gerada
- [ ] `ui/StatCard.jsx` — card de KPI
- [ ] `ui/EmptyState.jsx` — tela vazia com ação

### 4.3 — Layout

- [ ] `src/components/Layout.jsx` — sidebar + outlet + status WhatsApp

### 4.4 — Telas (nesta ordem)

- [ ] **Login** (`/login`) — autenticação com JWT, spinner, erro animado
- [ ] **Visão Geral** (`/`) — 4 StatCards + agenda do dia + alertas + atividade recente
- [ ] **Novo Cliente** (`/clientes/novo`) — formulário com validações e radio cards de plano
- [ ] **Clientes** (`/clientes`) — tabela com filtros, paginação, menu de ações, exportar CSV
- [ ] **Detalhe do Cliente** (`/clientes/:id`) — ficha completa com abas (histórico + notas)
- [ ] **Editar Cliente** (`/clientes/:id/editar`) — mesmo formulário do novo, pré-preenchido
- [ ] **Pagamentos** (`/pagamentos`) — histórico com 3 mini KPIs e filtros de período

### 4.5 — Hooks

- [ ] `src/hooks/useClientFilters.js` — filtros e paginação reutilizáveis

---

## Fase 5 — Integrações

> Referência: `05-INTEGRATIONS.md`

- [ ] Access Token de produção do Mercado Pago configurado no `.env`
- [ ] Webhook do MP cadastrado no painel: `URL/api/webhook/mercadopago`
- [ ] `MP_WEBHOOK_SECRET` configurado no `.env`
- [ ] `EVOLUTION_APIKEY`, `EVOLUTION_URL`, `EVOLUTION_INSTANCE` configurados
- [ ] `ADMIN_PHONE` configurado com número pessoal
- [ ] Mensagem de cobrança testada manualmente via dashboard
- [ ] Mensagem de confirmação testada via webhook simulado

---

## Fase 6 — Deploy

> Referência: `08-DEPLOY.md`

- [ ] Repositório `wapassist-api` commitado e no GitHub
- [ ] Repositório `wapassist-dashboard` commitado e no GitHub
- [ ] Backend deployado no Render com todas as variáveis de ambiente
- [ ] `GET https://wapassist-api.onrender.com/health` retorna `{ status: 'ok' }`
- [ ] Frontend deployado na Vercel com `VITE_API_URL` configurada
- [ ] Dashboard acessível pela URL da Vercel
- [ ] UptimeRobot configurado pingando `/health` a cada 5 minutos
- [ ] Webhook do MP atualizado para URL do Render em produção

---

## Fase 7 — Validação End-to-End

> Teste completo do fluxo antes de migrar os clientes reais.

- [ ] Login na dashboard em produção funcionando
- [ ] Cadastrar cliente de teste com telefone real
- [ ] Enviar cobrança manual — mensagem WhatsApp recebida
- [ ] Realizar pagamento Pix de teste
- [ ] Webhook processado — log no Render confirma
- [ ] Confirmação de pagamento recebida no WhatsApp do cliente de teste
- [ ] Data de vencimento atualizada corretamente na dashboard
- [ ] Alerta de inadimplência recebido no número admin (aguardar D+3 ou testar manualmente)

---

## Fase 8 — Calendário

> Referência: `06-CALENDAR.md`

- [ ] Tabelas `CalendarEvent` e `Recurrence` adicionadas ao schema Prisma
- [ ] `npx prisma db push` executado com o novo schema
- [ ] Rotas da API de eventos implementadas (`src/routes/events.js`)
- [ ] Serviço de recorrências implementado (`src/services/recurrence.js`)
- [ ] Job 3 (notificações a cada 5 min) adicionado ao scheduler
- [ ] Job 4 (expandir recorrências toda segunda) adicionado ao scheduler
- [ ] `sendEventNotification` implementado no `whatsapp.js`
- [ ] Dependências do frontend instaladas (FullCalendar + react-datepicker)
- [ ] Tela do calendário implementada com views mensal e semanal
- [ ] Rota `/calendario` adicionada ao `App.jsx` e à sidebar

---

## Fase 9 — IA (WhatsApp Inteligente)

> Referência: `07-PHASE-AI.md`  
> **Pré-requisito humano:** Conta OpenAI criada com créditos (ver `HUMAN-SETUP.md` Bloco 7)

- [ ] `OPENAI_API_KEY` e `MY_WHATSAPP` adicionados ao `.env` no Render
- [ ] `src/services/whisper.js` — transcrição de áudio implementada
- [ ] `src/services/aiAssistant.js` — todos os intents implementados
- [ ] `src/routes/whatsapp.js` — webhook de mensagens recebidas
- [ ] Webhook configurado na Evolution API apontando para o backend
- [ ] Teste: enviar "listar clientes" via WhatsApp → receber resposta
- [ ] Teste: enviar áudio com comando → receber resposta transcrita e executada
- [ ] Teste: criar lembrete via WhatsApp → receber notificação no horário

---

## Fase 10 — Migração dos Clientes Reais

- [ ] Listar todos os clientes atuais com nome, telefone, plano e data de vencimento
- [ ] Cadastrar todos os clientes na dashboard (pode ser feito em lotes via CSV — ver `11-IMPROVEMENTS.md` C1)
- [ ] Verificar se as datas de vencimento estão corretas
- [ ] Confirmar que o scheduler vai disparar cobranças na data correta
- [ ] Sistema em produção com clientes reais ✅

---

## Resumo de Progresso

| Fase | Status | Observações |
|---|---|---|
| Fase 0 — Pré-requisitos (`HUMAN-SETUP.md`) | ⬜ Pendente | |
| Fase 1 — Infraestrutura | ⬜ Pendente | |
| Fase 2 — Banco de Dados | ⬜ Pendente | |
| Fase 3 — Backend Core | ⬜ Pendente | |
| Fase 4 — Integrações | ⬜ Pendente | |
| Fase 5 — Frontend Base | ⬜ Pendente | |
| Fase 6 — Frontend Telas | ⬜ Pendente | |
| Fase 7 — Deploy + Validação | ⬜ Pendente | |
| Fase 8 — Calendário | ⬜ Pendente | |
| Fase 9 — IA (WhatsApp Inteligente) | ⬜ Pendente | |
| Fase 10 — Migração de Clientes | ⬜ Pendente | |

> Atualize a coluna **Status** com: ⬜ Pendente / 🔨 Em andamento / ✅ Concluído
