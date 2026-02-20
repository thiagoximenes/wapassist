# SETUP-PROGRESS — Estado Real da Infraestrutura

> **Última atualização:** 20/02/2026  
> **Para o agente:** Leia este arquivo no início de cada sessão para saber exatamente o que já foi feito e o que falta. Não repita etapas concluídas.

---

## Resumo Executivo

O setup de infraestrutura (Fase 0 + Fase 1) está **100% concluído**. O próximo passo é iniciar o desenvolvimento do backend (`wapassist-api`) a partir da Fase 2.

---

## Infraestrutura — O que está no ar

### VPS
- **Provedor:** Hostinger KVM 1
- **IP:** `72.61.57.129`
- **SO:** Ubuntu 22.04.5 LTS
- **Acesso SSH:** `ssh -i ~/.ssh/wapassist_vps root@72.61.57.129`
- **Chave SSH local:** `~/.ssh/wapassist_vps` (sem passphrase)
- **Senha root:** `<ver gerenciador de senhas>`

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
- **API Key:** `<ver gerenciador de senhas>`
- **Instância:** `wapassist`
- **Status:** ✅ Conectada (WhatsApp escaneado e funcionando)
- **Número conectado:** `5522992116841` (provisório — número de atendimento a clientes)
- **Docker Compose:** `/root/evolution/docker-compose.yml`
- **Nota técnica:** A v1.8.x retorna `state: null` no fetchInstances mas funciona corretamente — confirmado via envio de mensagem de teste bem-sucedido.
- **Formato de envio de mensagem (v1.8.x):**
  ```json
  POST /message/sendText/wapassist
  { "number": "5522999999999", "textMessage": { "text": "mensagem" } }
  ```

### DNS (Cloudflare — domínio yootiq.com)
| Registro | Tipo | Valor | Status |
|---|---|---|---|
| `apiwapassist.yootiq.com` | A | `72.61.57.129` | ✅ Propagado (DNS only) |
| `adminwapassist.yootiq.com` | CNAME | — | ⏳ Criar na Fase 10 (após Vercel) |

---

## Credenciais e Variáveis de Ambiente

### `.env` do backend (`wapassist-api`) — completo exceto MP_WEBHOOK_SECRET

```env
# ── Banco de dados ──────────────────────────────────────────────
DATABASE_URL=<ver gerenciador de senhas>

# ── Autenticação ────────────────────────────────────────────────
JWT_SECRET=<ver gerenciador de senhas>
ADMIN_PASSWORD=<ver gerenciador de senhas>

# ── Mercado Pago ────────────────────────────────────────────────
MP_ACCESS_TOKEN=<ver gerenciador de senhas — começa com APP_USR->
MP_WEBHOOK_SECRET=           ← PENDENTE: gerar no painel MP após backend no Render

# ── Evolution API (WhatsApp) ────────────────────────────────────
EVOLUTION_URL=https://apiwapassist.yootiq.com
EVOLUTION_APIKEY=<ver gerenciador de senhas>
EVOLUTION_INSTANCE=wapassist
ADMIN_PHONE=<seu número pessoal>

# ── OpenAI (Fase 8 — IA) ────────────────────────────────────────
OPENAI_API_KEY=<ver gerenciador de senhas — começa com sk-proj->
MY_WHATSAPP=<seu número pessoal>

# ── App ─────────────────────────────────────────────────────────
PORT=3000
FRONTEND_URL=https://adminwapassist.yootiq.com
NODE_ENV=production
```

### `.env.local` do frontend (`wapassist-dashboard`)

```env
VITE_API_URL=https://wapassist-api.onrender.com
```

### Mercado Pago
| Chave | Valor |
|---|---|
| Public Key | `<ver gerenciador de senhas>` |
| Access Token (produção) | `<ver gerenciador de senhas — começa com APP_USR->` |
| Client ID | `<ver gerenciador de senhas>` |
| Client Secret | `<ver gerenciador de senhas>` |

### UptimeRobot
| Chave | Valor |
|---|---|
| Main API Key | `<ver gerenciador de senhas>` |
| Read-only API Key | `<ver gerenciador de senhas>` |

---

## Contas e Serviços Externos

| Serviço | Status | Observação |
|---|---|---|
| **Hostinger VPS** | ✅ Contratado e configurado | KVM 1, ~R$20/mês |
| **Cloudflare** | ✅ DNS configurado | domínio `yootiq.com` |
| **Neon.tech** | ✅ Banco criado | projeto `neondb`, região `sa-east-1`, schema ainda não aplicado |
| **GitHub** | ✅ 3 repositórios criados | `wapassist` (docs), `wapassist-api` (vazio), `wapassist-dashboard` (vazio) |
| **Render.com** | ✅ Conta criada | Web Service ainda não criado |
| **Vercel** | ✅ Conta criada + MCP vinculado | Projeto ainda não criado |
| **UptimeRobot** | ✅ Conta criada | Monitor ainda não configurado |
| **Mercado Pago** | ✅ Credenciais de produção ativas | Webhook ainda não cadastrado |
| **OpenAI** | ✅ Conta criada, sem créditos | Adicionar créditos antes da Fase 8 |

---

## Repositórios GitHub

| Repositório | URL | Estado |
|---|---|---|
| `wapassist` | https://github.com/thiagoximenes/wapassist | Documentação — em uso |
| `wapassist-api` | https://github.com/thiagoximenes/wapassist-api | Vazio — próximo a ser desenvolvido |
| `wapassist-dashboard` | https://github.com/thiagoximenes/wapassist-dashboard | Vazio — Fase 5 |

---

## Próximos Passos (em ordem)

### Imediato — Fase 2 + 3 (backend)
1. Entrar na pasta `~/github/wapassist-api` (já clonada localmente)
2. Criar estrutura do projeto Node.js + Fastify + Prisma
3. Aplicar schema no Neon.tech (`npx prisma db push`)
4. Implementar todas as rotas do backend
5. Testar localmente
6. Push para GitHub + deploy no Render

### Após backend no ar — ações humanas necessárias
- [ ] Cadastrar webhook no painel Mercado Pago: `https://wapassist-api.onrender.com/api/webhook/mercadopago`
- [ ] Copiar o `MP_WEBHOOK_SECRET` gerado e adicionar nas env vars do Render
- [ ] Configurar monitor no UptimeRobot apontando para `https://wapassist-api.onrender.com/health`

### Fase 5+ (frontend)
- Entrar na pasta `~/github/wapassist-dashboard` (já clonada localmente)
- Criar projeto React + Vite + TailwindCSS
- Deploy na Vercel via MCP
- Criar registro CNAME `adminwapassist.yootiq.com` no Cloudflare

---

## Observações Técnicas Importantes

1. **Evolution API v1.8.x** — O formato de envio de mensagem é diferente das versões 2.x. Usar sempre `textMessage: { text: "..." }` e não `text: "..."` direto.
2. **Neon.tech connection string** — Inclui `channel_binding=require`. O Prisma aceita normalmente.
3. **Render free tier** — Dorme após 15 min de inatividade. O UptimeRobot (ping a cada 5 min) evita isso — configurar assim que o backend estiver no ar.
4. **WhatsApp número provisório** — O número `5522992116841` é o número de atendimento a clientes do Thiago. Substituir por chip dedicado assim que possível para evitar risco de ban.
5. **OpenAI sem créditos** — A API Key existe mas não funcionará até adicionar créditos. Não bloqueia as Fases 2-7.
