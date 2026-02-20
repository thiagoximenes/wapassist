# 10 — Solução de Problemas Comuns

> Referência rápida para os problemas mais frequentes durante o desenvolvimento e operação do Itaflix.

---

## Backend / API

### Render hiberna e perde webhooks do Mercado Pago

**Sintoma:** Pagamentos Pix são realizados mas o sistema não atualiza o cliente.  
**Causa:** O plano gratuito do Render hiberna após 15 minutos sem requisições. O webhook do MP chega durante a hibernação e é perdido.

**Solução:**
1. Configure o UptimeRobot para pingar `https://itaflix-api.onrender.com/health` a cada 5 minutos (ver `08-DEPLOY.md`)
2. Alternativa: upgrade para o plano Starter do Render (US$ 7/mês)

---

### Webhook do Mercado Pago não chega

**Sintoma:** Pix pago mas nenhum log no Render.  
**Causa:** URL do webhook incorreta ou backend inacessível.

**Diagnóstico:**
```bash
# Testar se o endpoint está acessível
curl -X POST https://itaflix-api.onrender.com/api/webhook/mercadopago \
  -H "Content-Type: application/json" \
  -d '{"type": "test"}'
# Esperado: { "ok": true }
```

**Soluções:**
- Confirmar que a URL no painel do MP está correta: `https://itaflix-api.onrender.com/api/webhook/mercadopago`
- Verificar se o backend está acordado (acessar `/health` primeiro)
- Verificar nos logs do Render se há erros de inicialização

---

### Cliente paga mas WhatsApp não envia confirmação

**Sintoma:** Webhook processado com sucesso (log mostra), mas cliente não recebe mensagem.  
**Causa:** Instância WhatsApp desconectada ou número formatado incorretamente.

**Diagnóstico:**
```bash
# Verificar status da instância
curl -X GET "https://api.itaflix.com.br/instance/fetchInstances" \
  -H "apikey: SUA_CHAVE"
# Verificar se state = "open"
```

**Soluções:**
- Se `state !== "open"`: reescanear o QR Code (ver `01-INFRASTRUCTURE.md`, Etapa 1.6)
- Verificar se o telefone do cliente está no formato `5521999998888` (sem espaços, traços ou parênteses)
- Verificar se `EVOLUTION_APIKEY`, `EVOLUTION_URL` e `EVOLUTION_INSTANCE` estão corretos no `.env`

---

### Data de vencimento calculada errada

**Sintoma:** Após pagamento, a nova data de vencimento está incorreta.  
**Causa:** Problema com fuso horário ou lógica de cálculo.

**Diagnóstico:**
```javascript
// Testar a função isoladamente (em um arquivo temporário)
import { calculateNewDueDate } from './src/services/billing.js';

// Caso 1: pagou antes do vencimento
const r1 = calculateNewDueDate('2025-03-15', '2025-03-14', 'MONTHLY');
console.log(r1); // Esperado: 2025-04-14

// Caso 2: pagou depois do vencimento
const r2 = calculateNewDueDate('2025-03-15', '2025-03-20', 'MONTHLY');
console.log(r2); // Esperado: 2025-04-19
```

**Soluções:**
- Garantir que `dueDate` está sendo salvo como `DATE` sem componente de hora
- Usar sempre UTC no backend — converter para `America/Sao_Paulo` apenas no frontend
- Verificar se o campo `paidAt` está sendo passado como `new Date()` (UTC) e não com fuso local

---

### Erro ao gerar link Pix no Mercado Pago

**Sintoma:** Log mostra erro `401` ou `invalid_token` ao tentar criar o link Pix.  
**Causa:** Access Token incorreto ou de sandbox.

**Soluções:**
- Confirmar que `MP_ACCESS_TOKEN` começa com `APP_USR-` (produção)
- Tokens de teste começam com `TEST-` e não funcionam para pagamentos reais
- Verificar se o token não expirou no painel do Mercado Pago

---

### Backend não inicia no Render

**Sintoma:** Deploy concluído mas serviço mostra erro.  
**Causa:** Variável de ambiente faltando ou erro no `prisma generate`.

**Diagnóstico:** Ver logs no painel do Render (aba **Logs**).

**Soluções:**
- Verificar se todas as variáveis de ambiente estão preenchidas no Render
- Confirmar que o Build Command inclui `npx prisma generate`
- Verificar se `DATABASE_URL` está correta e o banco Neon.tech está ativo

---

## Banco de Dados

### Neon.tech: banco suspenso

**Sintoma:** Erro de conexão com o banco após período de inatividade.  
**Causa:** O free tier do Neon.tech suspende após 7 dias sem atividade.

**Solução:**
- O UptimeRobot pingando o backend a cada 5 minutos mantém o banco ativo (o backend faz queries ao banco)
- Se já suspenso: acessar o painel do Neon.tech e reativar manualmente

---

### Erro de conexão SSL com o banco

**Sintoma:** `SSL connection required` ou `certificate verify failed`.  
**Causa:** `DATABASE_URL` sem o parâmetro `?sslmode=require`.

**Solução:**
```env
DATABASE_URL=postgresql://usuario:senha@host.neon.tech/itaflix?sslmode=require
```

---

### Prisma: tabela não encontrada

**Sintoma:** Erro `relation "Client" does not exist` ou similar.  
**Causa:** Schema não foi aplicado no banco.

**Solução:**
```bash
npx prisma db push
npx prisma generate
```

---

## WhatsApp / Evolution API

### Instância WhatsApp desconecta frequentemente

**Causa:** Celular sem bateria, sem internet, ou WhatsApp Web aberto no mesmo número.

**Soluções:**
- Manter o celular sempre carregado e com internet estável
- **Não usar** o mesmo número no WhatsApp Web enquanto a Evolution API estiver ativa
- Considerar usar um celular dedicado apenas para o Itaflix

---

### Container da Evolution API para após reinicialização da VPS

**Causa:** Docker não configurado para iniciar automaticamente.

**Solução:** O `restart: always` no `docker-compose.yml` garante reinício automático. Verificar se está presente:

```yaml
services:
  evolution-api:
    restart: always  # ← deve estar aqui
```

Se a VPS foi reiniciada e o container não subiu:
```bash
cd ~/evolution
docker compose up -d
```

---

### Mensagens não chegam para o cliente

**Sintoma:** Nenhum erro no log, mas o cliente não recebe.  
**Causa:** Número do cliente em formato incorreto.

**Formato correto:** `5521999998888` (código do país + DDD + número, sem espaços ou símbolos)

**Verificar no banco:**
```sql
SELECT phone FROM "Client" WHERE name = 'Nome do Cliente';
-- Deve retornar: 5521999998888
```

**Solução:** Atualizar o telefone do cliente na dashboard para o formato correto.

---

## Frontend

### Dashboard não carrega dados (tela em branco ou loading infinito)

**Causa:** `VITE_API_URL` incorreta ou backend hibernado.

**Diagnóstico:**
1. Abrir o DevTools do navegador (F12) → aba **Network**
2. Verificar se as requisições para a API estão retornando erro
3. Testar manualmente: `curl https://itaflix-api.onrender.com/health`

**Soluções:**
- Verificar se `VITE_API_URL` na Vercel aponta para a URL correta do Render
- Aguardar 30–60 segundos para o Render acordar (primeira requisição após hibernação)
- Verificar se o token JWT no `localStorage` não expirou (logout e login novamente)

---

### Erro 401 em todas as requisições após login

**Causa:** `JWT_SECRET` diferente entre o token gerado e o backend atual.

**Solução:**
- Verificar se `JWT_SECRET` no Render é o mesmo que foi usado ao gerar o token
- Fazer logout (limpa o token do `localStorage`) e login novamente

---

### Exportar CSV não funciona

**Causa:** Bloqueio de popup no navegador ou erro na geração do arquivo.

**Solução:**
- Permitir popups para o domínio da dashboard no navegador
- Verificar no console do navegador se há erros JavaScript

---

## Mercado Pago

### Taxa de transação inesperada

**Informação:** O Mercado Pago cobra ~0,99% por transação Pix aprovada. Para R$ 30 (plano mensal), a taxa é ~R$ 0,30. Isso é descontado automaticamente do valor recebido.

**Solução:** Considerar incluir a taxa no preço do plano ou absorvê-la como custo operacional.

---

### Pix gerado mas cliente não consegue pagar

**Causa:** Link Pix expirado (configurado para 48 horas).

**Solução:** Enviar nova cobrança manualmente via botão "Enviar cobrança" na dashboard do cliente.

---

## Checklist de Diagnóstico Rápido

Quando algo não funcionar, verificar nesta ordem:

1. [ ] Backend está acordado? → `curl https://itaflix-api.onrender.com/health`
2. [ ] WhatsApp conectado? → Verificar indicador na sidebar da dashboard
3. [ ] Banco ativo? → Tentar acessar a lista de clientes na dashboard
4. [ ] Variáveis de ambiente corretas? → Verificar no painel do Render
5. [ ] Logs do Render mostram algum erro? → Painel Render → Logs
6. [ ] Container Evolution API rodando? → `docker ps` na VPS
