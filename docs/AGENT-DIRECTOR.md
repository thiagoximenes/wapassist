# AGENT DIRECTOR — Guia Mestre de Desenvolvimento do wapassist

> **Para o agente de IA:** Este arquivo é a sua bússola. Leia-o integralmente antes de qualquer sessão de desenvolvimento. Ele define a ordem exata de implementação, quando testar, quando criar branches, quando pedir ação humana e onde a IA se encaixa no projeto.  
> **Para o humano:** Este arquivo define o que o agente fará em cada sessão. Antes de iniciar o coding, complete tudo em `HUMAN-SETUP.md`.

---

## Visão Geral das Fases

```
FASE 0 — Pré-requisitos humanos        → HUMAN-SETUP.md (você faz antes)
FASE 1 — Infraestrutura VPS            → branch: infra/vps-evolution
FASE 2 — Banco de dados                → branch: feat/database-schema
FASE 3 — Backend core                  → branch: feat/backend-core
FASE 4 — Integrações externas          → branch: feat/integrations
FASE 5 — Frontend base                 → branch: feat/frontend-base
FASE 6 — Frontend telas                → branch: feat/frontend-screens
FASE 7 — Calendário                    → branch: feat/calendar
FASE 8 — IA (WhatsApp inteligente)     → branch: feat/ai-assistant   ← integrada aqui
FASE 9 — Testes e hardening            → branch: test/e2e-validation
FASE 10 — Deploy produção              → merge → main + tag v1.0.0
```

---

## FASE 0 — Pré-requisitos Humanos

**Quem executa:** Você (humano)  
**Referência:** `HUMAN-SETUP.md`  
**Bloqueante:** Sim — nenhuma fase de código começa sem isso

O agente **não pode** criar contas, contratar VPS, escanear QR Code do WhatsApp ou gerar tokens em painéis externos. Tudo isso está detalhado em `HUMAN-SETUP.md`.

**Sinal de que está pronto:** Todas as variáveis do `.env.example` preenchidas no `.env` real.

---

## FASE 1 — Infraestrutura VPS + Evolution API

**Repositório:** nenhum (configuração de servidor)  
**Branch:** não se aplica (VPS é configurada via SSH)  
**Referência:** `01-INFRASTRUCTURE.md`

### Acesso SSH para o Agente

> ⚠️ **Importante:** O agente de IA **não tem acesso SSH direto** à VPS por padrão. Para que o agente execute comandos na VPS, você tem duas opções:

**Opção A — Você executa, agente instrui (recomendado para segurança):**
O agente gera os comandos exatos e você os cola no terminal SSH. Use `run_command` apenas para comandos locais.

**Opção B — Agente executa via `run_command` com SSH:**
Se você configurar uma chave SSH sem passphrase na máquina local apontando para a VPS, o agente pode executar:
```bash
ssh -i ~/.ssh/wapassist_vps root@IP_DA_VPS "comando aqui"
```
Para habilitar isso:
1. Gere a chave: `ssh-keygen -t ed25519 -f ~/.ssh/wapassist_vps -N ""`
2. Copie para a VPS: `ssh-copy-id -i ~/.ssh/wapassist_vps root@IP_DA_VPS`
3. Informe ao agente: "Use `ssh -i ~/.ssh/wapassist_vps root@IP_DA_VPS` para acessar a VPS"

**Ação humana obrigatória nesta fase:**
- [ ] Escanear QR Code do WhatsApp (só você pode fazer isso com o celular)

### O que o agente faz nesta fase
1. Gera todos os comandos de configuração da VPS em ordem
2. Configura `docker-compose.yml` da Evolution API
3. Gera o bloco de configuração do Nginx
4. Verifica a conexão via `curl https://api.wapassist.com.br`

### Teste de conclusão da fase
```bash
# Executar na VPS — deve retornar { state: "open" }
curl -X GET "https://api.wapassist.com.br/instance/fetchInstances" \
  -H "apikey: SUA_CHAVE"
```

---

## FASE 2 — Banco de Dados

**Repositório:** `wapassist-api`  
**Branch:** `feat/database-schema` → merge em `develop`  
**Referência:** `02-DATABASE.md`

### O que o agente faz
1. Cria o repositório `wapassist-api` com estrutura inicial
2. Escreve `prisma/schema.prisma` completo (MVP + Calendário + tabelas de IA)
3. Executa `npx prisma db push` e `npx prisma generate`
4. Cria `src/prisma.js` (singleton)

### Teste de conclusão da fase
```bash
npx prisma studio
# Verificar visualmente que todas as tabelas existem
```

### Merge
```bash
git checkout develop
git merge feat/database-schema
```

---

## FASE 3 — Backend Core

**Repositório:** `wapassist-api`  
**Branch:** `feat/backend-core` → merge em `develop`  
**Referência:** `03-BACKEND.md`

### Ordem de implementação dentro da fase

```
1. src/server.js              (entry point mínimo — só /health)
2. src/middleware/auth.js     (JWT verify)
3. src/services/billing.js    (regras de negócio — sem dependências externas)
4. src/routes/auth.js         (POST /api/auth/login)
5. src/routes/clients.js      (CRUD completo)
6. src/routes/notes.js        (notas por cliente)
7. src/routes/payments.js     (histórico — sem webhook ainda)
8. src/routes/dashboard.js    (GET /api/dashboard/summary)
9. src/routes/whatsapp.js     (GET /api/whatsapp/status)
```

### Testes intermediários (após cada grupo)
```bash
# Após item 4:
curl -X POST http://localhost:3000/api/auth/login -H "Content-Type: application/json" -d '{"password":"sua_senha"}'
# Esperado: { token: "eyJ..." }

# Após item 5:
curl -X GET http://localhost:3000/api/clients -H "Authorization: Bearer TOKEN"
# Esperado: []
```

### Teste de conclusão da fase
- `GET /health` → `{ status: 'ok' }`
- `POST /api/auth/login` → JWT válido
- CRUD de clientes funcionando localmente

### Merge
```bash
git checkout develop
git merge feat/backend-core
```

---

## FASE 4 — Integrações Externas

**Repositório:** `wapassist-api`  
**Branch:** `feat/integrations` → merge em `develop`  
**Referência:** `05-INTEGRATIONS.md`

### Ordem de implementação

```
1. src/services/whatsapp.js       (sendBillingReminder, sendPaymentConfirmation, sendOverdueNotice)
2. src/services/mercadopago.js    (createPixLink, getPayment)
3. src/routes/payments.js         (adicionar webhook POST /api/webhook/mercadopago)
4. src/services/scheduler.js      (Job 1 + Job 2)
5. src/routes/clients.js          (adicionar POST /api/clients/:id/send-billing)
```

### Ação humana obrigatória nesta fase
- [ ] Confirmar que o webhook do Mercado Pago está cadastrado com a URL do Render (só após deploy)

### Teste de conclusão da fase
```bash
# Teste manual do WhatsApp
curl -X POST http://localhost:3000/api/clients/1/send-billing \
  -H "Authorization: Bearer TOKEN"
# Verificar se mensagem chegou no WhatsApp do cliente de teste

# Teste do webhook (simular pagamento aprovado)
curl -X POST http://localhost:3000/api/webhook/mercadopago \
  -H "Content-Type: application/json" \
  -d '{"type":"payment","data":{"id":"ID_REAL_DO_MP"}}'
```

### Merge
```bash
git checkout develop
git merge feat/integrations
```

---

## FASE 5 — Frontend Base

**Repositório:** `wapassist-dashboard`  
**Branch:** `feat/frontend-base` → merge em `develop`  
**Referência:** `04-FRONTEND.md` (seções 4.1 a 4.4)

### O que o agente faz
1. Cria o projeto com Vite + React
2. Configura TailwindCSS + tokens CSS + fontes
3. Cria todos os componentes `ui/` (Badge, Button, Card, Input, etc.)
4. Cria `src/lib/api.js` e `src/lib/queryClient.js`
5. Cria `src/App.jsx` com roteamento e `PrivateRoute`
6. Cria `src/components/Layout.jsx` (sidebar + status WhatsApp)

### Teste via Playwright MCP
```
mcp0_browser_navigate → http://localhost:5173
mcp0_browser_snapshot → verificar se layout renderiza sem erros
mcp0_browser_console_messages → verificar se há erros JS
```

### Merge
```bash
git checkout develop
git merge feat/frontend-base
```

---

## FASE 6 — Frontend Telas

**Repositório:** `wapassist-dashboard`  
**Branch:** `feat/frontend-screens` → merge em `develop`  
**Referência:** `04-FRONTEND.md` (seções 4.5 a 4.13)

### Ordem de implementação das telas

```
1. LoginPage          (/login)           — sem dependência de API
2. HomePage           (/)                — GET /api/dashboard/summary
3. NewClientPage      (/clientes/novo)   — POST /api/clients
4. ClientsPage        (/clientes)        — GET /api/clients
5. ClientDetailPage   (/clientes/:id)    — GET /api/clients/:id
6. EditClientPage     (/clientes/:id/editar) — PUT /api/clients/:id
7. PaymentsPage       (/pagamentos)      — GET /api/payments
```

### Testes via Playwright MCP (por tela)

```
Login:
  navigate /login → fill senha → click Entrar → wait "Visão Geral"
  → screenshot → verificar redirecionamento

Novo cliente:
  navigate /clientes/novo → fill_form → click Salvar
  → wait confirmação → snapshot

Listagem:
  navigate /clientes → snapshot → verificar tabela e filtros
```

### Merge
```bash
git checkout develop
git merge feat/frontend-screens
```

---

## FASE 7 — Calendário

**Repositório:** `wapassist-api` + `wapassist-dashboard`  
**Branch:** `feat/calendar` em ambos → merge em `develop`  
**Referência:** `06-CALENDAR.md`

### Backend primeiro, depois frontend

**Backend:**
1. `src/routes/events.js` (CRUD de eventos)
2. `src/services/recurrence.js` (geração de ocorrências)
3. Adicionar Job 3 e Job 4 ao `scheduler.js`
4. Adicionar `sendEventNotification` ao `whatsapp.js`

**Frontend:**
1. Instalar FullCalendar + react-datepicker
2. `src/pages/CalendarPage.jsx` com views mensal e semanal
3. Modal de criação (único + recorrente)
4. Painel lateral de eventos do dia

### Merge
```bash
# Backend
git checkout develop && git merge feat/calendar

# Frontend
git checkout develop && git merge feat/calendar
```

---

## FASE 8 — IA (WhatsApp Inteligente)

> **Por que aqui?** A IA é implementada neste ponto porque:
> - O backend já tem todos os serviços (WhatsApp, billing, scheduler, clientes, notas, calendário)
> - Os intents da IA (`ADD_NOTE`, `REGISTER_PAYMENT`, `CREATE_REMINDER`, etc.) dependem de todos esses serviços já existirem e testados
> - Implementar antes geraria dependências circulares e dificuldade de debug

**Repositório:** `wapassist-api`  
**Branch:** `feat/ai-assistant` → merge em `develop`  
**Referência:** `07-PHASE-AI.md`

### Ação humana obrigatória nesta fase
- [ ] Criar conta na OpenAI e obter `OPENAI_API_KEY`
- [ ] Adicionar `OPENAI_API_KEY` e `MY_WHATSAPP` ao `.env` no Render

### O que o agente faz
1. `src/services/whisper.js` (transcrição de áudio)
2. `src/services/aiAssistant.js` (intents + GPT-4o Mini)
3. `src/routes/whatsapp.js` (webhook de mensagens recebidas)
4. Configurar webhook na Evolution API

### Teste de conclusão da fase
```
Enviar "listar clientes" via WhatsApp → receber resposta formatada
Enviar áudio com comando → receber resposta
Criar lembrete → receber notificação no horário
```

### Merge
```bash
git checkout develop
git merge feat/ai-assistant
```

---

## FASE 9 — Testes e Hardening

**Branch:** `test/e2e-validation` → merge em `develop`

### Testes automatizados via Playwright MCP

```
Fluxo 1 — Ciclo completo de pagamento:
  Cadastrar cliente → Enviar cobrança → Simular webhook → Verificar atualização

Fluxo 2 — Calendário:
  Criar tarefa recorrente → Verificar ocorrências → Marcar como feita

Fluxo 3 — IA:
  Enviar comando de texto → Verificar execução no banco

Fluxo 4 — Inadimplência:
  Forçar dueDate no passado → Rodar Job 2 manualmente → Verificar WhatsApp + dashboard
```

### Checklist de hardening
- [ ] Todas as rotas protegidas retornam 401 sem token
- [ ] Webhook do MP ignora eventos que não sejam `payment` + `approved`
- [ ] Scheduler não envia duplicatas (idempotência)
- [ ] Erros no WhatsApp não derrubam o fluxo principal (try/catch)
- [ ] `.env` não está commitado em nenhum repositório

---

## FASE 10 — Deploy de Produção

**Branch:** `develop` → merge em `main` → tag `v1.0.0`  
**Referência:** `08-DEPLOY.md`

### Ordem do deploy

```
1. Deploy backend no Render (via push para main do wapassist-api)
   → Verificar: GET https://wapassist-api.onrender.com/health

2. Deploy frontend na Vercel (via MCP vercel ou push para main do wapassist-dashboard)
   → Verificar: mcp0_browser_navigate → URL da Vercel → login

3. Configurar UptimeRobot (ação humana)

4. Cadastrar webhook do Mercado Pago com URL de produção (ação humana)

5. Escanear QR Code do WhatsApp em produção (ação humana)

6. Teste end-to-end em produção
```

### Merge final
```bash
git checkout main
git merge develop
git tag v1.0.0
git push origin main --tags
```

---

## Regras Permanentes para o Agente

1. **Sempre ler este arquivo** no início de cada sessão de desenvolvimento
2. **Sempre verificar em qual fase está** antes de escrever código
3. **Nunca pular fases** — cada fase tem dependências das anteriores
4. **Usar MCPs** conforme documentado em `MCP-TOOLS.md`
5. **Testar antes de fazer merge** — cada fase tem seu teste de conclusão
6. **Nunca commitar `.env`** — verificar `.gitignore` antes de qualquer `git push`
7. **Branches seguem a nomenclatura** definida neste arquivo — não inventar nomes
8. **Ações humanas** estão marcadas com `[ ]` — parar e avisar o usuário quando chegar nelas
9. **SSH na VPS** — perguntar ao usuário qual método prefere (Opção A ou B) antes de executar comandos remotos
10. **Erros de integração externa** (WhatsApp, MP) — nunca lançar exceção que derrube o servidor; sempre `try/catch` com log

---

## Estado Atual do Projeto

| Fase | Status |
|---|---|
| Fase 0 — Pré-requisitos | ✅ Concluído |
| Fase 1 — Infraestrutura | ✅ Concluído |
| Fase 2 — Banco de dados | 🔨 Em andamento (schema pendente) |
| Fase 3 — Backend core | ⬜ Pendente |
| Fase 4 — Integrações | ⬜ Pendente |
| Fase 5 — Frontend base | ⬜ Pendente |
| Fase 6 — Frontend telas | ⬜ Pendente |
| Fase 7 — Calendário | ⬜ Pendente |
| Fase 8 — IA | ⬜ Pendente |
| Fase 9 — Testes | ⬜ Pendente |
| Fase 10 — Deploy produção | ⬜ Pendente |

> Atualize esta tabela a cada sessão: ⬜ Pendente / 🔨 Em andamento / ✅ Concluído
