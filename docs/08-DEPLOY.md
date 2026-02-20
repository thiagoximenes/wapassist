# 08 — Deploy: Render (Backend) + Vercel (Frontend)

> **Objetivo:** Fazer o deploy do backend no Render e do frontend na Vercel, configurar variáveis de ambiente, CI/CD automático e manter o backend ativo 24/7.  
> **Tempo estimado:** 1–2 horas

---

## Visão Geral do Deploy

```
GitHub (wapassist-api)          →  Render (backend)
    git push main                  redeploy automático
                                   URL: https://wapassist-api.onrender.com

GitHub (wapassist-dashboard)    →  Vercel (frontend)
    git push main                  redeploy automático
                                   URL: https://admin.wapassist.com.br
```

---

## Etapa 8.1 — Preparar os Repositórios GitHub

Criar dois repositórios separados no GitHub: `wapassist-api` e `wapassist-dashboard`.

### Backend

```bash
cd wapassist-api
echo 'node_modules\n.env\n' > .gitignore
git init
git add .
git commit -m "feat: wapassist api inicial"
git remote add origin https://github.com/SEU_USUARIO/wapassist-api.git
git push -u origin main
```

### Frontend

```bash
cd wapassist-dashboard
echo 'node_modules\n.env.local\ndist\n' > .gitignore
git init
git add .
git commit -m "feat: wapassist dashboard inicial"
git remote add origin https://github.com/SEU_USUARIO/wapassist-dashboard.git
git push -u origin main
```

### Checklist

- [ ] Repositório `wapassist-api` criado no GitHub e código commitado
- [ ] Repositório `wapassist-dashboard` criado no GitHub e código commitado
- [ ] `.gitignore` configurado em ambos (sem `.env` e `node_modules`)

---

## Etapa 8.2 — Deploy do Backend no Render

1. Acesse [render.com](https://render.com) e crie uma conta gratuita
2. Clique em **New > Web Service**
3. Conecte ao repositório `wapassist-api` do GitHub
4. Configure:

| Campo | Valor |
|---|---|
| **Runtime** | Node |
| **Build Command** | `npm install && npx prisma generate` |
| **Start Command** | `node src/server.js` |
| **Plan** | Free |

5. Adicione todas as variáveis de ambiente (aba **Environment Variables**):

```env
DATABASE_URL          = (connection string do Neon.tech)
JWT_SECRET            = (openssl rand -hex 32)
ADMIN_PASSWORD        = (sua senha forte)
MP_ACCESS_TOKEN       = (do painel Mercado Pago — produção, começa com APP_USR-)
MP_WEBHOOK_SECRET     = (do painel Mercado Pago — webhooks)
EVOLUTION_URL         = https://api.wapassist.com.br
EVOLUTION_APIKEY      = (sua chave da Evolution API)
EVOLUTION_INSTANCE    = wapassist
ADMIN_PHONE           = (seu número: 5521999998888)
FRONTEND_URL          = https://admin.wapassist.com.br
PORT                  = 3000
```

6. Clique em **Create Web Service**
7. Aguarde o primeiro deploy (2–5 minutos)
8. Copie a URL pública gerada (ex: `https://wapassist-api.onrender.com`)
9. Cadastre essa URL como webhook no Mercado Pago:
   `https://wapassist-api.onrender.com/api/webhook/mercadopago`

### Verificar se o backend está funcionando

```bash
curl https://wapassist-api.onrender.com/health
# Esperado: { "status": "ok", "timestamp": "..." }
```

### Checklist

- [ ] Conta Render criada
- [ ] Web Service criado e conectado ao repositório `wapassist-api`
- [ ] Build Command: `npm install && npx prisma generate`
- [ ] Start Command: `node src/server.js`
- [ ] Todas as variáveis de ambiente configuradas no Render
- [ ] Primeiro deploy concluído sem erros
- [ ] `GET /health` retorna `{ status: 'ok' }`
- [ ] URL do webhook cadastrada no Mercado Pago

---

## Etapa 8.3 — Deploy do Frontend na Vercel

1. Acesse [vercel.com](https://vercel.com) e crie uma conta gratuita
2. Clique em **Add New > Project**
3. Importe o repositório `wapassist-dashboard` do GitHub
4. Framework Preset: **Vite** (detectado automaticamente)
5. Adicione a variável de ambiente:

```env
VITE_API_URL = https://wapassist-api.onrender.com
```

6. Clique em **Deploy**
7. Aguarde o deploy (1–3 minutos)
8. (Opcional) Em **Settings > Domains**, adicione `admin.wapassist.com.br`

### Configurar domínio customizado (opcional)

No painel do domínio `wapassist.com.br`, criar o registro:

| Tipo | Nome | Valor | TTL |
|---|---|---|---|
| CNAME | `admin` | `cname.vercel-dns.com` | 300 |

### Checklist

- [ ] Conta Vercel criada
- [ ] Projeto importado do repositório `wapassist-dashboard`
- [ ] `VITE_API_URL` configurada com a URL do Render
- [ ] Primeiro deploy concluído sem erros
- [ ] Dashboard acessível pela URL da Vercel
- [ ] Login com JWT funcionando em produção
- [ ] (Opcional) Domínio `admin.wapassist.com.br` configurado

---

## Etapa 8.4 — Manter o Backend Ativo (Plano Gratuito do Render)

> ⚠️ O plano gratuito do Render **hiberna após 15 minutos sem requisições**. Isso causa:
> - Demora de 30–60 segundos na primeira requisição após hibernação
> - Perda de webhooks do Mercado Pago durante a hibernação

### Solução: UptimeRobot (gratuito)

1. Acesse [uptimerobot.com](https://uptimerobot.com) e crie uma conta gratuita
2. Clique em **Add New Monitor**
3. Configure:

| Campo | Valor |
|---|---|
| Monitor Type | HTTP(S) |
| Friendly Name | wapassist API |
| URL | `https://wapassist-api.onrender.com/health` |
| Monitoring Interval | 5 minutes |

4. Salve — o UptimeRobot fará ping a cada 5 minutos, mantendo o backend acordado 24/7

> 💡 O mesmo ping também evita que o banco Neon.tech seja suspenso por inatividade.

### Alternativa: Plano pago do Render

Para produção com volume maior, o plano **Starter** do Render (US$ 7/mês) elimina a hibernação e oferece melhor performance.

### Checklist

- [ ] Conta UptimeRobot criada
- [ ] Monitor HTTP configurado para `https://wapassist-api.onrender.com/health`
- [ ] Intervalo de 5 minutos configurado
- [ ] Monitor ativo e fazendo pings

---

## Etapa 8.5 — CI/CD Automático

Após o setup inicial, o fluxo de trabalho para atualizar o sistema é:

```bash
# Fazer alteração no código
# ...

# Commitar e fazer push
git add .
git commit -m "feat: descrição da mudança"
git push origin main

# Render e Vercel fazem redeploy automático em 2-5 minutos
```

**Não é necessário nenhuma ação manual após o push.**

---

## Etapa 8.6 — Checklist de Validação em Produção

Após o deploy completo, validar o fluxo end-to-end:

- [ ] Acessar `https://admin.wapassist.com.br` e fazer login
- [ ] Cadastrar um cliente de teste
- [ ] Enviar cobrança manual via botão na dashboard
- [ ] Verificar se a mensagem chegou no WhatsApp do cliente de teste
- [ ] Realizar um pagamento Pix de teste
- [ ] Verificar se o webhook foi processado (log no Render)
- [ ] Verificar se a confirmação chegou no WhatsApp
- [ ] Verificar se a data de vencimento foi atualizada na dashboard

---

## Referência: URLs do Sistema em Produção

| Serviço | URL |
|---|---|
| Dashboard (admin) | `https://admin.wapassist.com.br` |
| Backend API | `https://wapassist-api.onrender.com` |
| Health check | `https://wapassist-api.onrender.com/health` |
| Webhook MP | `https://wapassist-api.onrender.com/api/webhook/mercadopago` |
| Evolution API | `https://api.wapassist.com.br` |
| Prisma Studio (local) | `http://localhost:5555` |

---

## Logs e Monitoramento

### Ver logs do backend no Render

No painel do Render, vá em **Logs** para ver os logs em tempo real.

### Ver logs da Evolution API na VPS

```bash
ssh root@IP_DA_VPS
docker logs evolution_api --tail 50 --follow
```

### Ver logs do banco no Neon.tech

No painel do Neon.tech, vá em **Monitoring** para ver queries e performance.
