# SETUP-PROGRESS — Estado Real do Sistema

> **Última atualização:** 20/02/2026  
> **Para o agente:** Leia este arquivo no início de cada sessão para saber exatamente o que já foi feito e o que falta. Não repita etapas concluídas.

---

## Resumo Executivo

Backend e frontend estão **100% implementados e funcionando localmente**. O próximo passo humano é fazer o deploy do backend no Render e do frontend na Vercel.

---

## Infraestrutura — O que está no ar

### VPS
- **Provedor:** Hostinger KVM 1
- **IP:** `72.61.57.129`
- **SO:** Ubuntu 22.04.5 LTS
- **Acesso SSH:** `ssh -i ~/.ssh/wapassist_vps root@72.61.57.129`
- **Chave SSH local:** `~/.ssh/wapassist_vps` (sem passphrase)

### Serviços instalados na VPS
| Serviço | Status | Versão |
|---|---|---|
| Docker | ✅ Rodando | latest |
| Nginx | ✅ Rodando | 1.18.0 |
| Certbot (SSL) | ✅ Configurado | auto-renovação ativa |
| Evolution API | ✅ Rodando | v1.8.7 (container `evolution_api`) |
| UFW Firewall | ✅ Ativo | portas 22, 80, 443 liberadas |

### Evolution API (WhatsApp)
- **URL pública:** `https://apiwapassist.yootiq.com`
- **Instância:** `wapassist`
- **Status:** ✅ Conectada (WhatsApp escaneado e funcionando)
- **Número conectado:** `5522992116841` (provisório)
- **Nota técnica:** v1.8.x retorna `state: null` no fetchInstances mas funciona — confirmado via envio de mensagem.
- **Formato de envio (v1.8.x):**
  ```json
  POST /message/sendText/wapassist
  { "number": "5522999999999", "textMessage": { "text": "mensagem" } }
  ```

### DNS (Cloudflare — domínio yootiq.com)
| Registro | Tipo | Valor | Status |
|---|---|---|---|
| `apiwapassist.yootiq.com` | A | `72.61.57.129` | ✅ Propagado |
| `adminwapassist.yootiq.com` | CNAME | — | ⏳ Criar após deploy Vercel |

---

## Repositórios GitHub

| Repositório | URL | Branch principal | Estado |
|---|---|---|---|
| `wapassist` | https://github.com/thiagoximenes/wapassist | `main` | Documentação — em uso |
| `wapassist-api` | https://github.com/thiagoximenes/wapassist-api | `develop` | ✅ Backend completo |
| `wapassist-dashboard` | https://github.com/thiagoximenes/wapassist-dashboard | `main` | ✅ Frontend completo |

---

## Backend — `wapassist-api` (branch `develop`)

### Stack
- Node.js v20, npm v10, ESM (`"type": "module"`)
- Fastify v5, @fastify/cors, @fastify/jwt
- Prisma v5 + Neon.tech PostgreSQL (região `sa-east-1`)
- mercadopago v2, node-cron v4, axios, date-fns

### Schema Prisma — Tabelas existentes
| Tabela | Descrição |
|---|---|
| `Client` | Clientes com nome, telefone, email, plano, dueDate, status, price |
| `Payment` | Pagamentos com paidAt, amount, newDueDate, mpPaymentId |
| `Note` | Notas livres por cliente |
| `CalendarEvent` | Eventos do calendário |
| `Recurrence` | Recorrências de eventos |
| `ClientLog` | Log de ações manuais (billing_sent, confirmation_sent) |

### Rotas implementadas
| Método | Rota | Descrição |
|---|---|---|
| POST | `/api/auth/login` | Login com senha → JWT |
| GET | `/api/auth/me` | Valida token |
| GET | `/api/clients` | Lista clientes (filtros: status, plan, search, page) |
| POST | `/api/clients` | Cria cliente |
| GET | `/api/clients/:id` | Detalhe do cliente (com payments e notes) |
| PUT | `/api/clients/:id` | Edita cliente |
| DELETE | `/api/clients/:id` | Remove cliente |
| POST | `/api/clients/:id/send-billing` | Envia cobrança WhatsApp + gera Pix + grava ClientLog |
| POST | `/api/clients/:id/send-confirmation` | Registra pagamento, recalcula vencimento, envia confirmação WhatsApp + grava ClientLog |
| GET | `/api/clients/:id/logs` | Logs de um cliente específico |
| GET | `/api/logs` | Todos os logs (filtro por type, limit) |
| GET | `/api/payments` | Histórico de pagamentos (com client aninhado) |
| POST | `/api/webhook/mercadopago` | Webhook MP — processa pagamento aprovado |
| GET | `/api/notes` | Lista notas |
| POST | `/api/notes` | Cria nota |
| DELETE | `/api/notes/:id` | Remove nota |
| GET | `/api/events` | Lista eventos do calendário |
| POST | `/api/events` | Cria evento |
| PUT | `/api/events/:id` | Edita evento |
| DELETE | `/api/events/:id` | Remove evento |
| GET | `/api/dashboard/summary` | KPIs para a home |
| GET | `/api/whatsapp/status` | Status da instância WhatsApp |
| GET | `/api/whatsapp/qrcode` | QR Code para reconectar |
| GET | `/health` | Health check |

### Serviços
- `billing.js` — `PLAN_DAYS` com 31 dias/mês fixo (MONTHLY=31, QUARTERLY=93, SEMIANNUAL=186, ANNUAL=372), `calculateNewDueDate`, `getPlanPrice`, `getPlanLabel`
- `whatsapp.js` — `sendBillingReminder`, `sendPaymentConfirmation`, `sendOverdueNotice`, `sendAdminAlert`, `sendEventNotification`
- `mercadopago.js` — `createPixLink`, `getPayment`
- `scheduler.js` — Job 1: cobrança D-1 às 09h | Job 2: inadimplência às 10h | Job 3: notificações de eventos a cada 5 min | Job 4: expandir recorrências toda segunda

### Lógica de recálculo de vencimento
- Pago **antes ou no dia** do vencimento → `newDueDate = dueDate + planDays`
- Pago **depois** do vencimento → `newDueDate = paidAt + planDays`
- Todos os meses têm **31 dias fixos**

---

## Frontend — `wapassist-dashboard` (branch `main`)

### Stack
- React 19 + Vite 7 + TailwindCSS 3
- react-router-dom v6, @tanstack/react-query, axios, date-fns, lucide-react
- Dark theme com CSS variables (tokens.css), fontes DM Sans + DM Mono

### Variáveis de ambiente
```env
# .env.local (desenvolvimento local)
VITE_API_URL=http://localhost:3000

# .env.example (produção — Vercel)
VITE_API_URL=https://wapassist-api.onrender.com
```

### Páginas implementadas
| Rota | Componente | Status |
|---|---|---|
| `/login` | `LoginPage` | ✅ |
| `/` | `HomePage` | ✅ KPIs, agenda do dia, alertas, atividade recente |
| `/clientes` | `ClientsPage` | ✅ Tabela com filtros, paginação, menu de ações |
| `/clientes/novo` | `NewClientPage` | ✅ Formulário com datetime-local para vencimento |
| `/clientes/:id` | `ClientDetailPage` | ✅ Ficha completa, botões icon-only com tooltip |
| `/clientes/:id/editar` | `NewClientPage` | ✅ Pré-preenchido |
| `/pagamentos` | `PaymentsPage` | ✅ Histórico com KPIs e filtros de período |
| `/calendario` | `CalendarPage` | ✅ |
| `/templates` | `TemplatesPage` | ✅ Templates editáveis com restore-to-default |
| `/logs` | `LogsPage` | ✅ Logs reais do DB, compacto, filtros |

### Componentes UI
`Badge`, `Button`, `Card`, `Input`, `Select`, `Modal`, `Toast`, `Skeleton`, `Avatar`, `StatCard`, `EmptyState`, `Tooltip`

### Funcionalidades implementadas
- **Autenticação** — JWT com interceptors axios, redirect automático no 401
- **Botões da página do cliente** — icon-only com tooltip; Editar (cinza), Enviar cobrança (amarelo `#facc15`/`#422006`), Enviar confirmação (verde `#4ade80`/`#052e16`)
- **Modal de confirmação** — abre antes de enviar cobrança ou confirmação
- **Templates** — sistema/padrão editáveis com localStorage, restore-to-default
- **Logs** — consome `GET /api/logs`, mostra billing_sent e confirmation_sent, sem duplicatas
- **Data + horário** — exibido em todos os campos de data (vencimento, cadastro, pagamentos)
- **Sidebar** — WhatsApp status com QR code modal, navegação completa
- **Avatar** — seguro contra name undefined/null/vazio

---

## Credenciais e Variáveis de Ambiente

### `.env` do backend (`wapassist-api`)
```env
DATABASE_URL=<ver gerenciador de senhas>
JWT_SECRET=<ver gerenciador de senhas>
ADMIN_PASSWORD=<ver gerenciador de senhas>
MP_ACCESS_TOKEN=<ver gerenciador de senhas>
MP_WEBHOOK_SECRET=<PENDENTE — gerar no painel MP após deploy Render>
EVOLUTION_URL=https://apiwapassist.yootiq.com
EVOLUTION_APIKEY=<ver gerenciador de senhas>
EVOLUTION_INSTANCE=wapassist
ADMIN_PHONE=<número pessoal>
OPENAI_API_KEY=<ver gerenciador de senhas>
MY_WHATSAPP=<número pessoal>
PORT=3000
FRONTEND_URL=https://adminwapassist.yootiq.com
NODE_ENV=production
```

---

## Contas e Serviços Externos

| Serviço | Status | Observação |
|---|---|---|
| **Hostinger VPS** | ✅ Configurado | KVM 1, Evolution API rodando |
| **Cloudflare** | ✅ DNS configurado | domínio `yootiq.com` |
| **Neon.tech** | ✅ Banco com schema aplicado | todas as tabelas criadas |
| **GitHub** | ✅ 3 repositórios com código | `wapassist-api` (develop), `wapassist-dashboard` (main) |
| **Render.com** | ⏳ Conta criada | **Deploy pendente — ação humana** |
| **Vercel** | ⏳ Conta criada + MCP vinculado | **Deploy pendente — ação humana** |
| **UptimeRobot** | ⏳ Conta criada | Monitor pendente (após Render) |
| **Mercado Pago** | ⏳ Credenciais ativas | Webhook pendente (após Render) |
| **OpenAI** | ⏳ Conta criada | Sem créditos — não bloqueia fases atuais |

---

## Ações Humanas Pendentes (em ordem)

1. **Render** — criar Web Service apontando para `wapassist-api` branch `main` (ou `develop`), adicionar todas as env vars do `.env`
2. **Mercado Pago** — cadastrar webhook: `https://wapassist-api.onrender.com/api/webhook/mercadopago`, copiar `MP_WEBHOOK_SECRET` gerado e adicionar no Render
3. **UptimeRobot** — criar monitor HTTP para `https://wapassist-api.onrender.com/health` a cada 5 min
4. **Vercel** — importar repositório `wapassist-dashboard`, definir `VITE_API_URL=https://wapassist-api.onrender.com`, fazer deploy
5. **Cloudflare** — criar CNAME `adminwapassist.yootiq.com` apontando para URL da Vercel
6. **WhatsApp** — re-escanear QR se necessário após Render deploy

---

## Observações Técnicas Importantes

1. **Evolution API v1.8.x** — usar `textMessage: { text: "..." }`, não `text: "..."` direto
2. **Neon.tech** — connection string inclui `channel_binding=require`, Prisma aceita normalmente
3. **Render free tier** — dorme após 15 min; UptimeRobot evita isso
4. **WhatsApp número provisório** — `5522992116841` é número de atendimento do Thiago; substituir por chip dedicado para evitar ban
5. **OpenAI** — API Key existe mas sem créditos; não bloqueia fases 2–7
6. **Prisma v5** — usar v5, não v6/v7 (incompatível com o setup atual)
7. **`ClientLog`** — tabela criada via `prisma db push` no branch `develop`; ao fazer merge para `main` no Render, rodar `npx prisma db push` novamente se necessário
