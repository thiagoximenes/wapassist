# wapassist — Visão Geral do Projeto

> **Sistema de Gestão de Assinaturas IPTV**  
> Dashboard web privada para gerenciar clientes, cobranças automáticas via Pix e notificações via WhatsApp.

---

## O que é o wapassist?

O wapassist é uma aplicação web **privada e de uso pessoal** para gerenciar uma operação de IPTV com ~30 clientes (meta: 100). O dono da operação é o único usuário da dashboard. O sistema automatiza:

- Cobrança via WhatsApp com link Pix (D-1 antes do vencimento)
- Confirmação de pagamento automática via webhook do Mercado Pago
- Atualização da data de vencimento do cliente
- Alertas de inadimplência para o admin

---

## Stack Tecnológica

| Camada | Tecnologia | Hospedagem | Custo |
|---|---|---|---|
| **Frontend** | React + Vite + TailwindCSS | Vercel | Gratuito |
| **Backend** | Node.js + Fastify + Prisma ORM | Render | Gratuito (free tier) |
| **Banco de Dados** | PostgreSQL | Neon.tech | Gratuito (0,5 GB) |
| **Pagamentos** | Mercado Pago Pix | — | ~0,99% por transação |
| **WhatsApp** | Evolution API (self-hosted) | VPS própria | ~R$ 20–25/mês |
| **Scheduler** | node-cron (embutido no backend) | — | Gratuito |
| **Autenticação** | JWT simples (só senha) | — | Gratuito |

---

## Custo Mensal Total

| Serviço | Custo |
|---|---|
| VPS Linux (Hostinger KVM 1 ou Hetzner CX11) | ~R$ 20–25/mês |
| React (Vercel) | R$ 0 |
| Node.js API (Render) | R$ 0 |
| PostgreSQL (Neon.tech) | R$ 0 |
| Evolution API (self-hosted na VPS) | R$ 0 |
| Mercado Pago (~0,99% por transação) | ~R$ 0,30 por cliente |
| **TOTAL estimado (30 clientes)** | **~R$ 30/mês** |

---

## Fluxo Completo do Sistema

```
[1] Admin cadastra cliente na dashboard
        ↓
[2] node-cron roda às 09h — identifica clientes com vencimento amanhã
        ↓
[3] Gera link Pix via Mercado Pago (external_reference = telefone do cliente)
        ↓
[4] Envia cobrança via WhatsApp (Evolution API)
        ↓
[5] Cliente paga o Pix
        ↓
[6] Mercado Pago dispara webhook para o backend
        ↓
[7] Backend identifica o cliente pelo external_reference
        ↓
[8] Calcula nova data de vencimento (regra de negócio — ver abaixo)
        ↓
[9] Atualiza banco de dados (status = ACTIVE, nova due_date)
        ↓
[10] Envia confirmação de pagamento via WhatsApp para o cliente
        ↓
[11] Dashboard atualiza em tempo real
```

---

## Regra de Negócio Central — Cálculo da Nova Data de Vencimento

Esta é a lógica mais crítica do sistema. **Nunca alterar sem testes.**

```
SE cliente pagou ANTES ou NO DIA do vencimento:
    nova_data = due_date_atual + dias_do_plano

SE cliente pagou DEPOIS do vencimento:
    nova_data = data_do_pagamento + dias_do_plano
```

### Dias por Plano

| Plano | Enum no banco | Dias | Preço padrão |
|---|---|---|---|
| Mensal | `MONTHLY` | 30 | R$ 30 |
| Trimestral | `QUARTERLY` | 90 | R$ 80 |
| Semestral | `SEMIANNUAL` | 180 | R$ 150 |
| Anual | `ANNUAL` | 365 | R$ 280 |

---

## Roadmap de Fases

| Fase | Status | Escopo |
|---|---|---|
| **MVP (Fase 1)** | ✅ Em produção (21/02/2026) | Dashboard CRUD, Pix automático, notificações WhatsApp |
| **Fase 2** | ⏳ Após MVP estável | Relatórios financeiros, histórico por cliente, múltiplos links Pix |
| **Fase 3 — IA** | 🔮 Futuro (3–6 meses) | Comandos por texto/voz no WhatsApp, assistente inteligente |

---

## Estrutura de Repositórios

O projeto é dividido em **dois repositórios GitHub separados**:

```
wapassist-api/          → Backend (Node.js + Fastify + Prisma)
wapassist-dashboard/    → Frontend (React + Vite + TailwindCSS)
```

---

## Domínio e Subdomínios

| Subdomínio | Destino | Função |
|---|---|---|
| `apiwapassist.yootiq.com` | VPS (Nginx → Evolution API porta 8080) | WhatsApp API ✅ |
| `adminwapassist.yootiq.com` | Vercel (frontend) | Dashboard admin ✅ |
| `wapassist-api.onrender.com` | Render (backend) | API REST + webhooks ✅ |

---

## Variáveis de Ambiente — Referência Rápida

Todas as variáveis necessárias para o backend (`.env`):

```env
# Banco de dados
DATABASE_URL=postgresql://...@ep-raspy-mud-acoglp71-pooler.sa-east-1.aws.neon.tech/neondb?sslmode=require
DIRECT_URL=postgresql://...@ep-raspy-mud-acoglp71.sa-east-1.aws.neon.tech/neondb?sslmode=require  # sem pooler, para migrations

# Autenticação
JWT_SECRET=<openssl rand -hex 32>
ADMIN_PASSWORD=<senha forte>
ADMIN_PHONE=<seu número: 5522997309370>

# Mercado Pago
MP_ACCESS_TOKEN=<token de PRODUÇÃO — começa com APP_USR->
MP_WEBHOOK_SECRET=<secret gerado no painel MP>

# Evolution API (WhatsApp)
EVOLUTION_URL=https://apiwapassist.yootiq.com
EVOLUTION_APIKEY=<chave gerada no docker-compose>
EVOLUTION_INSTANCE=wapassist

# App
PORT=3000
FRONTEND_URL=https://adminwapassist.yootiq.com
```

Variável necessária para o frontend (`.env.local`):

```env
VITE_API_URL=https://wapassist-api.onrender.com
```

---

## Índice da Documentação

| Arquivo | Conteúdo |
|---|---|
| `01-INFRASTRUCTURE.md` | VPS, Docker, Evolution API, SSL, WhatsApp QR Code |
| `02-DATABASE.md` | PostgreSQL no Neon.tech, schema completo, migrations |
| `03-BACKEND.md` | Node.js + Fastify, estrutura de arquivos, rotas, serviços, scheduler |
| `04-FRONTEND.md` | React dashboard, design system, todas as telas e componentes |
| `05-INTEGRATIONS.md` | Mercado Pago Pix, templates de mensagens WhatsApp |
| `06-CALENDAR.md` | Funcionalidade de calendário, recorrências, notificações |
| `07-PHASE-AI.md` | Camada de IA futura — WhatsApp inteligente com GPT + Whisper |
| `08-DEPLOY.md` | Deploy no Render e Vercel, uptime, CI/CD automático |
| `09-IMPLEMENTATION-CHECKLIST.md` | Checklist mestre de implementação por fase |
| `10-TROUBLESHOOTING.md` | Problemas conhecidos e soluções |
