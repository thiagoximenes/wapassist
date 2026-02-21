# 05 — Integrações: Mercado Pago + WhatsApp

> **Objetivo:** Configurar as integrações externas do sistema — geração de links Pix via Mercado Pago e envio de mensagens via Evolution API (WhatsApp).  
> **Tempo estimado:** 1–2 horas

---

## Parte A — Mercado Pago

### Etapa 5.1 — Criar Conta e Obter Credenciais

1. Acesse [mercadopago.com.br](https://mercadopago.com.br) e crie uma conta de **vendedor** (gratuita)
2. No painel, vá em **Seu negócio > Configurações > Credenciais**
3. Copie o **Access Token de PRODUÇÃO** (começa com `APP_USR-`)
4. Cole no campo `MP_ACCESS_TOKEN` do `.env` do backend

> ⚠️ **Nunca use o token de sandbox em produção.** Tokens de teste começam com `TEST-` e não processam pagamentos reais.

### Checklist

- [ ] Conta Mercado Pago de vendedor criada
- [ ] Access Token de **PRODUÇÃO** copiado (começa com `APP_USR-`)
- [ ] `MP_ACCESS_TOKEN` preenchido no `.env`

---

### Etapa 5.2 — Configurar Webhook no Mercado Pago

1. No painel MP, vá em **Seu negócio > Configurações > Notificações (Webhooks)**
2. Clique em **Adicionar nova URL de webhook**
3. URL: `https://wapassist-api.onrender.com/api/webhook/mercadopago`
4. Evento: **Pagamentos**
5. Salve e copie o **secret** gerado — ele vai para `MP_WEBHOOK_SECRET` no `.env`

> ⚠️ O backend precisa estar deployado no Render **antes** de configurar o webhook, pois o MP valida a URL ao salvar.

### Checklist

- [ ] Webhook cadastrado no painel do Mercado Pago
- [ ] URL aponta para `https://wapassist-api.onrender.com/api/webhook/mercadopago`
- [ ] Evento configurado: **Pagamentos**
- [ ] `MP_WEBHOOK_SECRET` copiado e preenchido no `.env`

---

### Etapa 5.3 — Como Funciona o PIX Nativo

O sistema utiliza **PIX nativo** do Mercado Pago, gerando QR Code diretamente na página de pagamento. O fluxo é:

```
[Scheduler D-1 ou Admin clica "Enviar cobrança"]
    │
    ├── Gera payToken único (6 chars alfanuméricos)
    │
    ├── Envia link via WhatsApp: FRONTEND_URL/pay/{payToken}
    │
    └── Cliente acessa página de pagamento
              │
              ├── GET /pay/:token → retorna dados do cliente
              │
              ├── Cliente clica "Gerar PIX"
              │
              ├── POST /pay/:token/pix → createPixPayment()
              │     ├── description: "wapassist - Renovação Mensal"
              │     ├── amount: 30.00
              │     ├── payment_method_id: "pix"
              │     ├── external_reference: "5521999998888"  ← telefone
              │     └── expiration: +24 horas
              │
              └── Retorna qrCode (copia-e-cola) + qrCodeBase64 (imagem)
```

**Fluxo de identificação no webhook:**

```
MP chama POST /api/webhook/mercadopago
    │
    ├── payment.external_reference = "5521999998888"
    │
    └── prisma.client.findUnique({ where: { phone: "5521999998888" } })
```

### Preços por Plano (configuráveis em `billing.js`)

| Plano | Enum | Valor padrão |
|---|---|---|
| Mensal | `MONTHLY` | R$ 30,00 |
| Trimestral | `QUARTERLY` | R$ 80,00 |
| Semestral | `SEMIANNUAL` | R$ 150,00 |
| Anual | `ANNUAL` | R$ 280,00 |

> 💡 Para alterar os preços, edite apenas `PLAN_PRICES` em `src/services/billing.js`. Não há necessidade de alterar o banco.

---

### Etapa 5.4 — Testar o Webhook Manualmente

Para testar se o backend processa o webhook corretamente:

```bash
curl -X POST https://wapassist-api.onrender.com/api/webhook/mercadopago \
  -H "Content-Type: application/json" \
  -d '{"type": "payment", "data": {"id": "ID_DO_PAGAMENTO_REAL"}}'
```

Para simular um pagamento aprovado em sandbox (ambiente de testes):

```bash
# Use o token de TESTE para simular
MP_ACCESS_TOKEN=TEST-... node -e "
  const { MercadoPagoConfig, Payment } = require('mercadopago');
  // ... criar pagamento de teste
"
```

---

## Parte B — WhatsApp (Evolution API)

### Etapa 5.5 — Templates de Mensagens

Todos os templates são definidos em `src/services/whatsapp.js`. Abaixo estão os templates completos para referência e ajuste.

---

#### Template 1 — Cobrança D-1

**Enviada automaticamente 1 dia antes do vencimento (Job 1 do scheduler)**

```
Olá, *{NOME}*! 👋

Sua assinatura *wapassist* vence amanhã, *{DATA}*.

Para renovar, acesse o link abaixo e gere seu PIX:
🔗 {FRONTEND_URL}/pay/{PAYTOKEN}

O link expira em 48 horas.
Qualquer dúvida é só chamar! 😊
```

---

#### Template 2 — Confirmação de Pagamento

**Enviada automaticamente após detecção do Pix no webhook**

```
✅ *Pagamento confirmado!*

Olá, *{NOME}*!
Recebemos seu pagamento com sucesso.

📅 Nova validade: *{NOVA_DATA}*
📦 Plano: {PLANO}

Bom entretenimento! 🎬
— Equipe wapassist
```

---

#### Template 3 — Aviso de Inadimplência (para o Cliente)

**Enviada para o WhatsApp do cliente quando ele passa 3 dias em atraso**

```
⏰ *wapassist — Assinatura em atraso*

Olá, *{NOME}*!

Sua assinatura está em atraso há *{DIAS} dias* (venceu em {DATA_VENCIMENTO}).

Para regularizar, entre em contato ou aguarde a próxima cobrança automática.

Qualquer dúvida é só chamar! 😊
— Equipe wapassist
```

> ℹ️ O admin **não recebe WhatsApp** nesse evento. A notificação para o admin é exibida apenas na **dashboard** (painel de alertas da tela Visão Geral), consumida via `GET /api/dashboard/summary` → campo `alerts`.

---

#### Template 4 — Cobrança Manual (acionada pelo admin na dashboard)

**Enviada quando o admin clica em "Enviar cobrança" na dashboard**

Mesmo template da Cobrança D-1, mas disparado manualmente via `POST /api/clients/:id/send-billing`.

---

### Rotas de Pagamento Público

#### GET /pay/:token

Rota **pública** (sem autenticação JWT) que retorna informações do cliente para exibição na página de pagamento.

**Resposta:**
```json
{
  "name": "João Silva",
  "plan": "MONTHLY",
  "planLabel": "Mensal",
  "amount": 30.00,
  "status": "ACTIVE",
  "dueDate": "2026-03-15",
  "daysUntilDue": 24
}
```

#### POST /pay/:token/pix

Rota **pública** que gera o PIX nativo no Mercado Pago.

**Resposta:**
```json
{
  "mpPaymentId": "123456789",
  "qrCode": "00020126580014br.gov.bcb.pix...",
  "qrCodeBase64": "data:image/png;base64,iVBORw0KGgo...",
  "expiresAt": "2026-02-22T10:30:00.000Z",
  "amount": 30.00
}
```

---

### Etapa 5.6 — Verificar Status da Instância

A sidebar da dashboard exibe o status da conexão WhatsApp em tempo real. O backend expõe:

```
GET /api/whatsapp/status
→ { connected: true | false }
```

Se `connected: false`, o admin precisa reescanear o QR Code na VPS:

```bash
# Na VPS, deletar a sessão atual
curl -X DELETE "https://api.wapassist.com.br/instance/logout/wapassist" \
  -H "apikey: SUA_CHAVE"

# Recriar e obter novo QR Code
curl -X POST "https://api.wapassist.com.br/instance/connect/wapassist" \
  -H "apikey: SUA_CHAVE"
```

---

### Etapa 5.7 — Boas Práticas para Não Ser Banido

| Prática | Motivo |
|---|---|
| Usar número dedicado (não pessoal) | Evita perder o número pessoal em caso de ban |
| Não enviar mensagens em massa para números desconhecidos | O WhatsApp detecta spam |
| Manter o celular com bateria e internet estável | Instância desconecta sem conexão |
| Não usar WhatsApp Web no mesmo número | Conflito de sessão |
| Mensagens personalizadas (nome do cliente) | Reduz chance de ser marcado como spam |
| Limitar a ~50 mensagens/hora em horário comercial | Evita detecção de automação |

---

## Checklist Geral das Integrações

- [ ] Conta Mercado Pago de vendedor criada
- [ ] Access Token de produção configurado no `.env`
- [ ] Webhook do MP cadastrado e apontando para o backend
- [ ] `MP_WEBHOOK_SECRET` configurado no `.env`
- [ ] Evolution API rodando na VPS (ver `01-INFRASTRUCTURE.md`)
- [ ] Instância `wapassist` criada e WhatsApp conectado (status `open`)
- [ ] `EVOLUTION_APIKEY`, `EVOLUTION_URL`, `EVOLUTION_INSTANCE` configurados no `.env`
- [ ] `ADMIN_PHONE` configurado com seu número pessoal
- [ ] Mensagem de teste enviada com sucesso via WhatsApp
- [ ] Teste end-to-end: Link enviado → Página aberta → PIX gerado → Pago → Webhook processado → WhatsApp enviado
- [ ] Página de pagamento `/pay/:token` funcionando sem autenticação
- [ ] QR Code exibido corretamente (copia-e-cola + imagem base64)
