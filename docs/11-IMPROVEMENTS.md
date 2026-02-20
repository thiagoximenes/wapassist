# 11 — Melhorias e Funcionalidades Adicionais

> Funcionalidades profissionais que agregam valor real ao wapassist, organizadas por esforço de implementação e impacto no negócio. O agente deve consultar este arquivo ao planejar sprints futuros.

---

## Categoria A — Alto Impacto, Baixo Esforço
> Implementar logo após o MVP estável

### A1 — Renovação com Um Clique via WhatsApp

**Problema:** O cliente recebe o link Pix, mas precisa abrir o navegador para pagar.  
**Solução:** Incluir na mensagem de cobrança um botão de resposta rápida "Já paguei ✅". Quando o cliente responde, o sistema registra o pagamento manualmente (sem webhook) e envia confirmação.

**Impacto:** Reduz fricção para clientes que pagam em dinheiro ou transferência direta.  
**Implementação:** Webhook de mensagens recebidas (já previsto na Fase 8) + intent `CONFIRM_PAYMENT_MANUAL`.

---

### A2 — Histórico de Notificações Enviadas

**Problema:** Não há como saber se uma mensagem WhatsApp foi realmente enviada ou falhou.  
**Solução:** Criar tabela `NotificationLog` no banco registrando cada mensagem enviada (tipo, destinatário, status, timestamp).

```prisma
model NotificationLog {
  id        Int      @id @default(autoincrement())
  clientId  Int?
  type      String   // BILLING_REMINDER, PAYMENT_CONFIRM, OVERDUE_NOTICE, etc.
  phone     String
  status    String   // SENT, FAILED
  error     String?
  sentAt    DateTime @default(now())
  client    Client?  @relation(fields: [clientId], references: [id])
}
```

**Impacto:** Auditoria completa, debug de falhas, confiança no sistema.  
**Implementação:** Wrapper na função `sendMessage` do `whatsapp.js`.

---

### A3 — Filtro de Clientes por Vencimento Hoje / Esta Semana

**Problema:** O admin precisa ver rapidamente quem vence hoje para agir.  
**Solução:** Adicionar filtros rápidos na tela de Clientes: "Vence hoje", "Vence esta semana", "Vencidos".  
**Implementação:** Parâmetro `?duePeriod=today|week|overdue` na rota `GET /api/clients`.

---

### A4 — Reenvio Automático de Cobrança (D+1 após vencimento)

**Problema:** Clientes que não pagaram no dia do vencimento não recebem novo lembrete.  
**Solução:** Job adicional no scheduler: no dia seguinte ao vencimento, reenviar cobrança com tom mais urgente.

```
Job 5 — Recobrança D+1
Horário: 10h diário
Condição: dueDate = ontem AND status = ACTIVE (não pagou ainda)
Mensagem: template diferente, mais direto
```

---

### A5 — Dashboard Mobile-Friendly

**Problema:** A dashboard atual é otimizada para desktop.  
**Solução:** Adicionar breakpoints responsivos no TailwindCSS. Sidebar vira menu hambúrguer em telas < 768px. Tabelas viram cards empilhados no mobile.  
**Impacto:** Admin pode gerenciar pelo celular quando estiver fora de casa.

---

## Categoria B — Alto Impacto, Médio Esforço
> Implementar após o MVP estar em produção há pelo menos 2 semanas

### B1 — Relatório Financeiro Mensal (PDF)

**Problema:** Não há como exportar um relatório do mês para controle pessoal ou contabilidade.  
**Solução:** Endpoint `GET /api/reports/monthly?month=2&year=2026` que gera um PDF com:
- Total recebido no mês
- Lista de pagamentos com data, cliente, plano e valor
- Clientes inadimplentes
- Comparativo com mês anterior

**Tecnologia:** `pdfkit` ou `puppeteer` no backend.  
**Frontend:** Botão "Exportar PDF" na tela de Pagamentos.

---

### B2 — Múltiplos Preços por Cliente (Preço Customizado)

**Problema:** Alguns clientes podem ter preços negociados individualmente.  
**Solução:** Adicionar campo `customPrice` opcional na tabela `Client`. Se preenchido, usa esse valor ao gerar o Pix; se nulo, usa o preço padrão do plano.

```prisma
model Client {
  // ... campos existentes
  customPrice Decimal? @db.Decimal(10, 2)
}
```

---

### B3 — Notificação de Aniversário de Cliente

**Problema:** Oportunidade de fidelização perdida.  
**Solução:** Campo `birthDate` opcional no cadastro do cliente. Job no scheduler verifica aniversários do dia e envia mensagem personalizada via WhatsApp.

```
🎂 Feliz aniversário, *{NOME}*!
A equipe wapassist deseja um ótimo dia para você! 🎉
```

---

### B4 — Painel de Saúde do Sistema

**Problema:** Admin não sabe se os jobs do scheduler estão rodando corretamente.  
**Solução:** Tela `/sistema` na dashboard mostrando:
- Status do WhatsApp (conectado/desconectado)
- Último horário de execução de cada job
- Total de mensagens enviadas hoje
- Erros nas últimas 24h (do `NotificationLog`)
- Status do banco (ping)
- Uptime do servidor

**Backend:** Endpoint `GET /api/system/health` com todas essas métricas.

---

### B5 — Backup Automático dos Dados

**Problema:** Neon.tech free tier não garante backup automático.  
**Solução:** Job semanal que exporta todos os dados em JSON e envia para um bucket S3 (ou Cloudflare R2 — gratuito até 10 GB).

**Alternativa mais simples:** Script que faz `pg_dump` e salva em pasta local na VPS com rotação de 30 dias.

---

### B6 — Link de Autoatendimento para o Cliente

**Problema:** Cliente precisa falar com o admin para saber a data de vencimento ou pedir segunda via.  
**Solução:** Página pública `https://admin.wapassist.com.br/cliente/:token` onde o cliente vê:
- Data de vencimento atual
- Plano ativo
- Botão para gerar novo link Pix

O `token` é gerado por hash do telefone — sem necessidade de login.

---

## Categoria C — Médio Impacto, Médio Esforço
> Implementar conforme demanda

### C1 — Importação em Massa de Clientes (CSV Upload)

**Problema:** Migrar 30+ clientes manualmente é trabalhoso.  
**Solução:** Tela de importação que aceita CSV com colunas `nome,telefone,plano,vencimento`. O sistema valida, mostra preview e importa em lote.

---

### C2 — Tags/Grupos de Clientes

**Problema:** Não há como segmentar clientes (ex: "região sul", "indicados por fulano").  
**Solução:** Tabela `Tag` com relação N:N com `Client`. Filtro por tag na listagem.

---

### C3 — Histórico de Alterações (Audit Log)

**Problema:** Não há rastreabilidade de quem alterou o quê.  
**Solução:** Tabela `AuditLog` registrando toda alteração em `Client` (campo alterado, valor anterior, valor novo, timestamp).

---

### C4 — Integração com Google Calendar

**Problema:** Admin já usa Google Calendar pessoal.  
**Solução:** Sincronizar eventos do calendário wapassist com Google Calendar via OAuth2 + Google Calendar API. Cobranças e tarefas aparecem no Google Calendar do admin.

---

### C5 — Modo Escuro / Claro

**Problema:** O sistema só tem tema dark.  
**Solução:** Toggle de tema com persistência no `localStorage`. Tema claro com as mesmas variáveis CSS redefinidas.

---

## Categoria D — Funcionalidades de IA Avançadas (Fase 8+)

### D1 — Resumo Diário Automático via WhatsApp

Todo dia às 08h, o sistema envia para o admin um resumo:

```
📊 *Resumo wapassist — Quinta, 20/02*

✅ Ativos: 28 clientes
⏰ Vencem hoje: 3
🔴 Em atraso: 2 (Carlos Lima, Ana Costa)
💰 Recebido ontem: R$ 90

Bom dia! 🌅
```

**Implementação:** Job 5 no scheduler + template de resumo diário.

---

### D2 — Análise de Inadimplência com IA

O GPT analisa o histórico de pagamentos de cada cliente e classifica o risco:
- 🟢 Pagador pontual
- 🟡 Pagador irregular
- 🔴 Alto risco de churn

Exibido como badge na ficha do cliente e na listagem.

---

### D3 — Sugestão de Plano Ideal

Com base no histórico de pagamentos, a IA sugere ao admin qual plano seria mais vantajoso para cada cliente (ex: "João sempre paga em dia — considere oferecer plano anual com desconto").

---

### D4 — Chatbot de Suporte para o Cliente

O cliente pode enviar mensagens para o número do wapassist e receber respostas automáticas:
- "Quando vence minha assinatura?"
- "Como pago?"
- "Preciso de ajuda"

Respostas automáticas para perguntas frequentes; escala para o admin quando não souber responder.

---

## Prioridade de Implementação Sugerida

```
MVP estável
    ↓
A2 (log de notificações) — base para debug
A1 (confirmação manual) — reduz trabalho do admin
A4 (recobrança D+1) — reduz inadimplência
A3 (filtros rápidos) — melhora UX
A5 (mobile) — acesso pelo celular
    ↓
B4 (painel de saúde) — visibilidade operacional
B1 (relatório PDF) — controle financeiro
B2 (preço customizado) — flexibilidade comercial
B6 (autoatendimento cliente) — reduz suporte manual
    ↓
D1 (resumo diário) — fácil, alto valor
D2 (análise de risco) — inteligência de negócio
```
