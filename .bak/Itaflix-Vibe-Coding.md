
**ITAFLIX**
Guia de Vibe Coding
Prompts, contexto e comandos para construir o MVP com Cursor, Windsurf ou qualquer IDE com IA
|**Cursor**|**Windsurf**|**Qualquer IDE + IA**|
| :-: | :-: | :-: |
Complemento do Itaflix MVP v2.0  •  Use este arquivo junto com o guia principal

# **Como Usar Este Guia**
Este arquivo é um complemento prático ao Itaflix MVP v2.0. Em vez de explicar o que construir, ele te diz exatamente o que falar para a IA do seu IDE construir por você.
Cada seção contém:
- Um prompt de contexto inicial — cole uma vez ao abrir o projeto
- Prompts de tarefas específicas — cole quando quiser gerar cada parte do código
- Comandos de terminal — copie e execute sem digitar
- Dicas de revisão — o que checar depois que a IA gerar o código
**💡  Vibe Coding funciona melhor quando você dá contexto rico. Nunca peça só 'crie um backend'. Os prompts deste guia já incluem stack, regras de negócio e restrições para a IA gerar código funcional de primeira.**
```text

```

## **IDEs Recomendados**
|**IDE**|**Modelo padrão**|**Melhor para**|**Custo**|
| :- | :- | :- | :- |
|Cursor|Claude 3.5 / GPT-4o|Edição e refatoração de arquivos existentes|US$ 20/mês (ou free limitado)|
|Windsurf|Claude 3.5 Sonnet|Geração de projetos do zero, fluxo mais fluido|Gratuito com créditos|
|VS Code + Copilot|GPT-4o|Quem já usa VS Code, integração nativa|US$ 10/mês|
|Zed + Claude|Claude 3.5 Sonnet|Editor rápido, bom para backend|Gratuito (editor)|
**💡  Recomendação: comece com o Windsurf (gratuito) para criar o projeto do zero, depois use o Cursor para manutenção e iterações.**
```text

```
|**01**| **Prompt de Contexto Master**
 Cole isto UMA VEZ ao abrir o projeto no IDE — define tudo
|
| :-: | :- |
Abra o Cursor ou Windsurf, crie uma pasta vazia chamada itaflix e cole o prompt abaixo na janela de chat da IA (Ctrl+L no Cursor, Cmd+L no Windsurf):
**🤖  PROMPT MASTER — Cole uma vez ao iniciar o projeto**
```text

```
| Você vai me ajudar a construir o Itaflix, um sistema de gestão de assinaturas de IPTV.
 Sou o único usuário do sistema (dono da operação). Tenho ~30 clientes hoje, meta de 100.
 === STACK DEFINIDA (não sugira alternativas) ===
 - Frontend: React + Vite + TailwindCSS (deploy na Vercel)
 - Backend: Node.js + Fastify + Prisma ORM (deploy no Render)
 - Banco: PostgreSQL hospedado no Neon.tech
 - Pagamentos: Mercado Pago (webhook automático de Pix)
 - WhatsApp: Evolution API (self-hosted, já configurada em https://api.itaflix.com.br)
 - Scheduler: node-cron integrado ao backend
 - Auth: JWT simples (só eu acesso a dashboard)
 === REGRAS DE NEGÓCIO CRÍTICAS ===
 Planos disponíveis: monthly (30d), quarterly (90d), semiannual (180d), annual (365d)
 Cálculo de nova data de vencimento após pagamento:
   SE pagou ANTES ou NO DIA do vencimento:
     nova\_data = due\_date\_atual + dias\_do\_plano
   SE pagou DEPOIS do vencimento:
     nova\_data = data\_do\_pagamento + dias\_do\_plano
 Notificações automáticas via WhatsApp:
   - D-1 antes do vencimento: envia cobrança com link Pix gerado pelo Mercado Pago
   - Após pagamento confirmado: envia confirmação com nova data de vencimento
   - D+3 após vencimento sem pagamento: envia alerta para MEU número pessoal
 === VARIÁVEIS DE AMBIENTE (já existem no .env) ===
 DATABASE\_URL, JWT\_SECRET, ADMIN\_PASSWORD, MP\_ACCESS\_TOKEN, MP\_WEBHOOK\_SECRET,
 EVOLUTION\_URL, EVOLUTION\_APIKEY, EVOLUTION\_INSTANCE, ADMIN\_PHONE, FRONTEND\_URL
 === PREFERÊNCIAS DE CÓDIGO ===
 - Português nos comentários e mensagens de log
 - Inglês nos nomes de variáveis, funções e arquivos
 - Sempre trate erros com try/catch e log descritivo
 - Use async/await, nunca callbacks
 - Prisma para todas as queries ao banco (nunca SQL puro)
 - Confirme o que vai criar ANTES de escrever o código
|
**✅  Após colar o prompt master, a IA tem todo o contexto do projeto. A partir daí, use os prompts das seções seguintes para construir cada parte.**
```text

```
|**02**| **Backend — Prompts e Comandos**
 Node.js + Fastify + Prisma + Neon.tech
|
| :-: | :- |

## **2.1 — Criar o Projeto e Instalar Dependências**
Abra o terminal integrado do IDE (Ctrl+` no Cursor/Windsurf) e execute:
**bash**
```bash

```
| mkdir itaflix-api && cd itaflix-api
 npm init -y
 npm install fastify @fastify/cors @fastify/jwt dotenv
 npm install @prisma/client mercadopago node-cron axios
 npm install -D prisma nodemon
 npx prisma init --datasource-provider postgresql
|

## **2.2 — Gerar o Schema do Banco com Prisma**
Abra o arquivo prisma/schema.prisma e cole o prompt abaixo no chat da IA:
**🤖  PROMPT — Schema Prisma completo**
```text

```
| Escreva o schema Prisma completo para o arquivo prisma/schema.prisma do Itaflix.
 Inclua os models:
 Client:
   id, name, phone (único), email (opcional), plan (enum: MONTHLY/QUARTERLY/SEMIANNUAL/ANNUAL),
   status (enum: ACTIVE/INACTIVE/OVERDUE), dueDate, createdAt, updatedAt
   Relações: payments[], notes[]
 Payment:
   id, clientId (FK), mpPaymentId (opcional, string), amount, paidAt, newDueDate, createdAt
 Note:
   id, clientId (FK), content, createdAt
 Gere também o arquivo .env com DATABASE\_URL vazio para eu preencher.
 Use nomes de campos em camelCase no schema.
|
Após a IA gerar o schema, execute:
**bash**
```bash

```
| # Preencha DATABASE\_URL no .env com a string do Neon.tech antes de rodar
 npx prisma db push
 npx prisma studio
|
**✅  O Prisma Studio abre no navegador em localhost:5555 e mostra as tabelas visualmente. Confirme que clients, payments e notes foram criadas.**
```text

```

## **2.3 — Estrutura de Arquivos do Backend**
**🤖  PROMPT — Criar estrutura de pastas e arquivos vazios**
```text

```
| Crie a estrutura de pastas e arquivos do projeto itaflix-api conforme abaixo.
 Crie os arquivos vazios agora. Vamos preencher cada um separadamente.
 itaflix-api/
 ├── src/
 │   ├── server.js
 │   ├── prisma.js              (singleton do PrismaClient)
 │   ├── routes/
 │   │   ├── auth.js
 │   │   ├── clients.js
 │   │   ├── payments.js
 │   │   └── notes.js
 │   ├── services/
 │   │   ├── whatsapp.js
 │   │   ├── mercadopago.js
 │   │   ├── billing.js         (cálculo de datas)
 │   │   └── scheduler.js
 │   └── middleware/
 │       └── auth.js
 ├── prisma/
 │   └── schema.prisma
 ├── .env
 └── package.json
|

## **2.4 — Gerar o Servidor Principal**
**🤖  PROMPT — src/server.js**
```text

```
| Escreva o arquivo src/server.js do Itaflix com Fastify.
 Deve:
 - Registrar @fastify/cors (origin: process.env.FRONTEND\_URL)
 - Registrar @fastify/jwt (secret: process.env.JWT\_SECRET)
 - Importar e registrar todas as rotas de src/routes/ com prefixo /api
 - Importar e iniciar o scheduler de src/services/scheduler.js
 - Escutar na porta process.env.PORT ou 3000
 - Ter um endpoint GET /health que retorna { status: 'ok', timestamp: new Date() }
 - Log de inicialização em português: 'Servidor Itaflix rodando na porta X'
|

## **2.5 — Serviço de WhatsApp**
**🤖  PROMPT — src/services/whatsapp.js**
```text

```
| Escreva o arquivo src/services/whatsapp.js do Itaflix.
 Usa axios para chamar a Evolution API.
 Variáveis de ambiente: EVOLUTION\_URL, EVOLUTION\_APIKEY, EVOLUTION\_INSTANCE
 Exporte 3 funções:
 1\. sendBillingReminder(phone, name, dueDate, pixLink)
    Mensagem de cobrança D-1. Formato:
    Olá, \*{name}\*! Sua assinatura Itaflix vence amanhã, \*{data formatada DD/MM/YYYY}\*.
    Para renovar: {pixLink}
    Válido por 48 horas.
 2\. sendPaymentConfirmation(phone, name, newDueDate, plan)
    Mensagem de confirmação. Inclui nova data e nome do plano em português.
 3\. sendAdminAlert(message)
    Envia mensagem para process.env.ADMIN\_PHONE.
    Usada para alertas de cliente em atraso e erros críticos.
 Todas as funções devem ter try/catch com log descritivo.
 Se a chamada falhar, logar o erro mas NÃO lançar exceção (não pode derrubar o fluxo principal).
|

## **2.6 — Lógica de Cobrança e Datas**
**🤖  PROMPT — src/services/billing.js**
```text

```
| Escreva src/services/billing.js com as funções de cálculo do Itaflix.
 Exporte:
 1\. calculateNewDueDate(currentDueDate, paidAt, plan)
    Plan é string: 'MONTHLY' | 'QUARTERLY' | 'SEMIANNUAL' | 'ANNUAL'
    Dias por plano: MONTHLY=30, QUARTERLY=90, SEMIANNUAL=180, ANNUAL=365
    Regra:
      Se paidAt <= currentDueDate: nova = currentDueDate + dias
      Se paidAt > currentDueDate:  nova = paidAt + dias
    Retorna objeto Date.
 2\. getPlanPrice(plan)
    Retorna o valor em reais de cada plano.
    Use valores padrão: MONTHLY=30, QUARTERLY=80, SEMIANNUAL=150, ANNUAL=280.
    (Esses valores podem ser ajustados depois)
 3\. getPlanLabel(plan)
    Retorna o nome em português: Mensal, Trimestral, Semestral, Anual.
 Inclua testes unitários simples no final do arquivo (comentados) para validar a lógica das datas.
|

## **2.7 — Webhook do Mercado Pago**
**🤖  PROMPT — src/routes/payments.js (webhook)**
```text

```
| Escreva src/routes/payments.js do Itaflix com Fastify.
 Rotas:
 POST /webhook/mercadopago (sem autenticação JWT — chamado pelo Mercado Pago)
   - Valide o webhook com MP\_WEBHOOK\_SECRET no header x-signature
   - Se type !== 'payment', retorne 200 imediatamente
   - Busque o pagamento na API do Mercado Pago pelo data.id
   - Se status !== 'approved', retorne 200 imediatamente
   - Busque o cliente pelo phone = payment.external\_reference
   - Se não encontrar: chame whatsapp.sendAdminAlert com os detalhes e retorne 200
   - Calcule nova data com billing.calculateNewDueDate
   - Salve o pagamento no banco (tabela Payment)
   - Atualize client.dueDate e client.status = ACTIVE
   - Chame whatsapp.sendPaymentConfirmation
   - Retorne { ok: true }
 GET /payments (com autenticação JWT)
   - Retorna todos os pagamentos com dados do cliente, ordenados por paidAt desc
   - Aceita query param ?clientId=X para filtrar por cliente
 Importe: prisma de ../prisma.js, billing de ../services/billing.js,
 whatsapp de ../services/whatsapp.js, mercadopago de ../services/mercadopago.js
|

## **2.8 — CRUD de Clientes**
**🤖  PROMPT — src/routes/clients.js**
```text

```
| Escreva src/routes/clients.js do Itaflix com Fastify.
 Todas as rotas exigem autenticação JWT (use o middleware de auth).
 Rotas:
 GET /clients
   Query params opcionais: status, plan, search (busca por nome ou telefone)
   Retorna clientes ordenados por dueDate asc (quem vence primeiro aparece primeiro)
   Inclui campo 'daysUntilDue' calculado (pode ser negativo se estiver em atraso)
 GET /clients/:id
   Retorna cliente com payments[] e notes[] incluídos
 POST /clients
   Body: name, phone, email?, plan, dueDate
   Valide: phone único, plan válido, dueDate é uma data válida
   Formate o phone removendo caracteres não numéricos e garantindo formato 55DDNÚMERO
 PUT /clients/:id
   Permite atualizar: name, phone, email, plan, dueDate, status
   Não permita mudar id ou createdAt
 DELETE /clients/:id
   Soft delete: apenas muda status para INACTIVE, não apaga do banco
 POST /clients/:id/send-billing
   Gera um link Pix pelo Mercado Pago e envia cobrança manual via WhatsApp
   Retorna { sent: true, pixLink }
|

## **2.9 — Scheduler de Cobranças Automáticas**
**🤖  PROMPT — src/services/scheduler.js**
```text

```
| Escreva src/services/scheduler.js do Itaflix com node-cron.
 Implemente 2 jobs:
 JOB 1: Cobrança D-1 (roda todo dia às 09:00 horário de Brasília, cron: '0 9 \* \* \*')
   - Busca clientes com dueDate = amanhã E status = ACTIVE
   - Para cada cliente:
     - Gera link Pix com mercadopago.createPixLink
     - Envia cobrança com whatsapp.sendBillingReminder
     - Loga: 'Cobrança enviada para {name} - vence em {data}'
 JOB 2: Alerta de inadimplência (roda todo dia às 10:00, cron: '0 10 \* \* \*')
   - Busca clientes com dueDate < hoje - 3 dias E status != INACTIVE
   - Atualiza status desses clientes para OVERDUE
   - Envia um único resumo para ADMIN\_PHONE com a lista de inadimplentes
   - Formato do resumo: '⚠️ Clientes em atraso: {lista com nome e dias de atraso}'
 Exporte função startScheduler() que registra os dois jobs.
 Log ao iniciar: 'Scheduler Itaflix iniciado. Jobs: cobrança D-1 (09h) e inadimplência (10h)'
|
|**03**| **Frontend — Prompts da Dashboard React**
 React + Vite + TailwindCSS
|
| :-: | :- |

## **3.1 — Criar e Configurar o Projeto**
**bash**
```bash

```
| npm create vite@latest itaflix-dashboard -- --template react
 cd itaflix-dashboard
 npm install
 npm install axios react-router-dom @tanstack/react-query date-fns
 npm install lucide-react
 npm install -D tailwindcss postcss autoprefixer
 npx tailwindcss init -p
|
Após instalar, abra o projeto no IDE e cole o prompt abaixo para configurar o TailwindCSS:
**🤖  PROMPT — Configurar Tailwind + estrutura inicial**
```text

```
| Configure o TailwindCSS no projeto React/Vite do Itaflix:
 1\. Atualize tailwind.config.js com content: ['./index.html', './src/\*\*/\*.{js,jsx}']
 2\. Adicione as diretivas @tailwind no src/index.css
 3\. Crie src/lib/api.js com uma instância do axios apontando para
    VITE\_API\_URL (variável de ambiente), com interceptor que adiciona
    o JWT token do localStorage em todas as requisições
 4\. Crie src/lib/queryClient.js com a configuração do React Query
 5\. Atualize src/main.jsx para envolver o App com QueryClientProvider e BrowserRouter
|

## **3.2 — Sistema de Rotas e Layout**
**🤖  PROMPT — Rotas e layout da aplicação**
```text

```
| Crie o sistema de rotas do Itaflix em src/App.jsx e o layout principal.
 Rotas:
   /login              → LoginPage (pública)
   /                   → redireciona para /clientes
   /clientes           → ClientsPage (protegida)
   /clientes/novo      → NewClientPage (protegida)
   /clientes/:id       → ClientDetailPage (protegida)
   /pagamentos         → PaymentsPage (protegida)
 Layout para rotas protegidas (src/components/Layout.jsx):
   - Sidebar à esquerda com navegação (ícones + labels do lucide-react)
   - Logo ITAFLIX no topo da sidebar em roxo
   - Links: Clientes, Pagamentos
   - Botão de logout no rodapé da sidebar
   - Área principal à direita com header mostrando o título da página atual
   - Design escuro/moderno com cores: fundo #0F172A, sidebar #1E293B, accent #7C3AED
 PrivateRoute: se não tiver JWT no localStorage, redireciona para /login
|

## **3.3 — Tela de Login**
**🤖  PROMPT — src/pages/LoginPage.jsx**
```text

```
| Crie a tela de login do Itaflix em src/pages/LoginPage.jsx.
 Visual:
   - Tela centralizada, fundo escuro (#0F172A)
   - Card com logo ITAFLIX em roxo, campo de senha, botão Entrar
   - Loading state no botão enquanto faz a requisição
   - Mensagem de erro em vermelho se a senha estiver errada
 Lógica:
   - POST /api/auth/login com { password }
   - Salva o token JWT retornado no localStorage como 'itaflix\_token'
   - Redireciona para /clientes após login bem-sucedido
 Não precisa de campo de usuário — só senha.
|

## **3.4 — Tela Principal de Clientes**
**🤖  PROMPT — src/pages/ClientsPage.jsx**
```text

```
| Crie a tela de listagem de clientes do Itaflix em src/pages/ClientsPage.jsx.
 Cards de resumo no topo (4 cards lado a lado):
   - Total de clientes ativos
   - Vencem esta semana (daysUntilDue entre 0 e 7)
   - Em atraso (status OVERDUE)
   - Receita estimada do mês (soma dos preços dos planos dos clientes ativos)
 Filtros acima da tabela:
   - Input de busca por nome ou telefone
   - Select de status: Todos, Ativo, Em atraso, Inativo
   - Select de plano: Todos, Mensal, Trimestral, Semestral, Anual
 Tabela de clientes com colunas:
   Nome | Telefone | Plano | Vencimento | Status | Ações
   - Vencimento: mostrar em vermelho se em atraso, amarelo se vence em até 3 dias
   - Status: badge colorido (verde=Ativo, vermelho=Em atraso, cinza=Inativo)
   - Ações: botão Ver detalhes e botão Enviar cobrança (ícone de WhatsApp)
 Botão Novo Cliente no canto superior direito que navega para /clientes/novo.
 Use React Query para buscar os dados. Loading skeleton nas linhas da tabela.
|

## **3.5 — Formulário de Novo Cliente**
**🤖  PROMPT — src/pages/NewClientPage.jsx**
```text

```
| Crie o formulário de cadastro de novo cliente em src/pages/NewClientPage.jsx.
 Campos:
   - Nome completo (obrigatório)
   - Telefone (obrigatório) — máscara: (99) 99999-9999, salva como 55DDNÚMERO
   - E-mail (opcional)
   - Plano: select com Mensal (R$30), Trimestral (R$80), Semestral (R$150), Anual (R$280)
   - Data de vencimento inicial: date picker
 Ao selecionar o plano, mostre abaixo: 'Próxima renovação em X dias por R$ Y'
 Botões: Cancelar (volta para /clientes) e Cadastrar cliente
 Validações client-side antes de enviar:
   - Telefone com 11 dígitos
   - Data de vencimento não pode ser no passado
 POST /api/clients — ao sucesso, navega para /clientes com toast de confirmação.
|

## **3.6 — Detalhe do Cliente**
**🤖  PROMPT — src/pages/ClientDetailPage.jsx**
```text

```
| Crie a tela de detalhes do cliente em src/pages/ClientDetailPage.jsx.
 Busca os dados via GET /api/clients/:id (inclui payments e notes).
 Layout em duas colunas:
 Coluna esquerda — Dados do cliente:
   - Card com nome, telefone, e-mail, plano, status
   - Botão Editar que transforma os campos em inputs editáveis (inline edit)
   - Botão Salvar / Cancelar quando em modo edição
   - Botão Enviar cobrança agora (chama POST /api/clients/:id/send-billing)
   - Botão Desativar cliente (muda status para INACTIVE)
 Coluna direita — em abas:
   Aba Histórico de pagamentos:
     Tabela com: data do pagamento, valor, nova data de vencimento gerada
   Aba Notas:
     Lista de notas com data
     Textarea + botão Adicionar nota (POST /api/notes com clientId)
 No topo: breadcrumb Clientes > {nome do cliente}
|
|**04**| **Deploy — Comandos e Configurações**
 Render (backend) + Vercel (frontend)
|
| :-: | :- |

## **4.1 — Preparar o Repositório**
Antes do deploy, crie dois repositórios separados no GitHub: itaflix-api e itaflix-dashboard.
**bash**
```bash

```
| # No backend
 cd itaflix-api
 echo 'node\_modules\n.env\nprisma/\*.db' > .gitignore
 git init && git add . && git commit -m 'feat: itaflix api inicial'
 git remote add origin https://github.com/SEU\_USUARIO/itaflix-api.git
 git push -u origin main
 # No frontend
 cd itaflix-dashboard
 echo 'node\_modules\n.env.local\ndist' > .gitignore
 git init && git add . && git commit -m 'feat: itaflix dashboard inicial'
 git remote add origin https://github.com/SEU\_USUARIO/itaflix-dashboard.git
 git push -u origin main
|

## **4.2 — Deploy do Backend no Render**
1. Acesse render.com > New > Web Service
1. Conecte o repositório itaflix-api
1. Configure:
   - Runtime: Node
   - Build Command: npm install && npx prisma generate
   - Start Command: node src/server.js
   - Plan: Free
1. Adicione as variáveis de ambiente (copie do seu .env):
**.env**
```text

```
| DATABASE\_URL          = (connection string do Neon.tech)
 JWT\_SECRET            = (openssl rand -hex 32)
 ADMIN\_PASSWORD        = (sua senha forte)
 MP\_ACCESS\_TOKEN       = (do painel Mercado Pago — produção)
 MP\_WEBHOOK\_SECRET     = (do painel Mercado Pago — webhooks)
 EVOLUTION\_URL         = https://api.itaflix.com.br
 EVOLUTION\_APIKEY      = (sua chave da Evolution API)
 EVOLUTION\_INSTANCE    = itaflix
 ADMIN\_PHONE           = (seu número: 5521999998888)
 FRONTEND\_URL          = https://itaflix-dashboard.vercel.app
 PORT                  = 3000
|
1. Após o deploy, copie a URL do Render (ex: https://itaflix-api.onrender.com)
1. Cadastre essa URL como webhook no Mercado Pago: URL/webhook/mercadopago

## **4.3 — Deploy do Frontend na Vercel**
1. Acesse vercel.com > Add New > Project
1. Importe o repositório itaflix-dashboard
1. Framework Preset: Vite (detectado automaticamente)
1. Adicione a variável de ambiente:
**.env**
```text
VITE\_API\_URL = https://itaflix-api.onrender.com
```
1. Clique em Deploy
1. (Opcional) Em Settings > Domains, adicione admin.itaflix.com.br
**💡  Toda vez que você fizer git push para o main, o Render e a Vercel fazem redeploy automático. Seu fluxo de trabalho será: edita no IDE → git push → deploy automático.**
```text

```

## **4.4 — Manter o Render Ativo (Plano Gratuito)**
O Render gratuito hiberna após 15 minutos sem requisições. Configure um ping periódico gratuito:
1. Acesse uptimerobot.com e crie uma conta gratuita
1. Novo monitor > HTTP(S)
1. URL: https://itaflix-api.onrender.com/health
1. Intervalo: 5 minutos
1. Ativo — isso mantém o backend acordado 24/7 sem custo
|**05**| **Dicas de Vibe Coding Avançadas**
 Como extrair o máximo dos IDEs com IA
|
| :-: | :- |

## **5.1 — Prompts de Correção e Iteração**
Quando a IA gerar algo que não ficou certo, use estes prompts de refinamento:
**🤖  PROMPT — Quando o código tem um bug específico**
```text

```
| O código que você gerou para [NOME\_DO\_ARQUIVO] tem um problema:
 [DESCREVA O COMPORTAMENTO ERRADO]
 O comportamento esperado é:
 [DESCREVA O QUE DEVERIA ACONTECER]
 Não reescreva o arquivo inteiro. Mostre apenas a função/trecho que precisa mudar.
|
**🤖  PROMPT — Quando a IA mudou algo que não devia**
```text

```
| Você alterou [PARTE QUE NÃO DEVIA SER ALTERADA] que estava funcionando.
 Reverta apenas essa parte para o estado anterior.
 Mantenha a correção que fizemos em [PARTE QUE DEVIA SER ALTERADA].
|
**🤖  PROMPT — Adicionar funcionalidade sem quebrar o que existe**
```text

```
| Preciso adicionar [FUNCIONALIDADE] ao arquivo [ARQUIVO].
 Não altere nenhuma função existente.
 Adicione apenas o código novo necessário.
 Mostre exatamente onde no arquivo o novo código deve ser inserido.
|

## **5.2 — Prompts de Revisão e Qualidade**
**🤖  PROMPT — Revisão de segurança antes do deploy**
```text

```
| Revise o arquivo [ARQUIVO] buscando:
 1\. Rotas sem autenticação que deveriam ter
 2\. Dados sensíveis expostos na resposta da API (senhas, tokens)
 3\. SQL injection ou manipulação de dados sem validação
 4\. Variáveis de ambiente hardcoded no código
 5\. Erros não tratados que poderiam derrubar o servidor
 Liste os problemas encontrados com a linha exata e como corrigir.
 Se não encontrar problemas, confirme que cada ponto foi verificado.
|
**🤖  PROMPT — Gerar arquivo .env.example para documentação**
```text

```
| Analise todos os arquivos do projeto e gere um arquivo .env.example
 com todas as variáveis de ambiente usadas.
 Para cada variável, adicione um comentário explicando o que é e onde obter.
 Use valores fictícios/placeholder, nunca os valores reais.
|

## **5.3 — Fluxo de Trabalho Recomendado**
|**#**|**Etapa**|**O que fazer**|
| :- | :- | :- |
|**1**|**Abrir o projeto**|Abra a pasta no Cursor/Windsurf. Cole o Prompt Master (Seção 01) uma única vez.|
|**2**|**Pedir o scaffold**|Use o prompt da seção correspondente para criar os arquivos. Revise a estrutura antes de prosseguir.|
|**3**|**Testar localmente**|npm run dev no frontend, node src/server.js no backend. Teste no browser antes de seguir.|
|**4**|**Iterar com prompts de correção**|Se algo não funcionar, use os prompts da Seção 5.1. Seja específico sobre o erro.|
|**5**|**Commit e push**|git add . && git commit -m 'feat: descrição' && git push — deploy automático na Vercel/Render.|
|**6**|**Validar em produção**|Acesse a URL da Vercel e teste o fluxo completo: cadastrar cliente → enviar cobrança → confirmar pagamento.|
|**06**| **Fase IA — Prompts para o Futuro**
 Quando o MVP estiver pronto e você quiser expandir
|
| :-: | :- |
Guarde estes prompts para quando o MVP estiver funcionando em produção. Eles constroem a camada de IA por cima do sistema que já existe.

## **6.1 — Receber e Interpretar Mensagens do WhatsApp**
**🤖  PROMPT FUTURO — Webhook de mensagens recebidas**
```text

```
| Adicione ao backend do Itaflix um endpoint para receber mensagens do WhatsApp.
 POST /webhook/whatsapp (sem JWT — chamado pela Evolution API)
   - A Evolution API envia mensagens recebidas neste endpoint
   - Configure o webhook na Evolution API apontando para esta URL
   - Extraia: sender (phone), messageType (text/audio), content
   - Se messageType = audio: transcreva com OpenAI Whisper API
   - Envie o texto (transcrito ou direto) para a função processCommand(text, senderPhone)
   - processCommand usa OpenAI GPT-4o Mini para identificar a intenção e responder
 Variáveis novas no .env:
   OPENAI\_API\_KEY = (chave da OpenAI)
   MY\_WHATSAPP = (seu número — só aceita comandos do SEU número)
 Se a mensagem vier de outro número que não MY\_WHATSAPP, ignore silenciosamente.
|
**🤖  PROMPT FUTURO — Função processCommand com IA**
```text

```
| Crie src/services/aiAssistant.js no Itaflix.
 Função: processCommand(userMessage, senderPhone)
   Chama GPT-4o Mini com:
   - System prompt: 'Você é o assistente do Itaflix. Extraia a intenção e retorne JSON.
     Intenções possíveis: ADD\_NOTE, LIST\_CLIENTS, REGISTER\_PAYMENT, CREATE\_REMINDER,
     LIST\_OVERDUE, UNKNOWN.
     Retorne: { intent, clientName?, phone?, content?, date?, time? }'
   - User message: o texto recebido
   Com base na intenção retornada, execute a ação correspondente:
   - ADD\_NOTE: busca cliente por nome, salva nota no banco
   - LIST\_CLIENTS: retorna lista formatada dos clientes com vencimento próximo
   - REGISTER\_PAYMENT: busca cliente, registra pagamento manual
   - CREATE\_REMINDER: salva agendamento, cron verifica a cada minuto
   - LIST\_OVERDUE: retorna clientes em atraso
   - UNKNOWN: responde 'Não entendi. Tente: listar clientes, anotar algo sobre [cliente]...'
   Sempre responda via whatsapp.sendAdminAlert com o resultado da ação.
|

## **6.2 — Sistema de Lembretes Pessoais**
**🤖  PROMPT FUTURO — Tabela de reminders e scheduler**
```text

```
| Adicione suporte a lembretes pessoais no Itaflix.
 1\. Adicione ao schema Prisma:
    model Reminder {
      id, message, scheduledAt (DateTime), sent (Boolean default false), createdAt
    }
 2\. No scheduler.js, adicione um job que roda a cada minuto ('\* \* \* \* \*'):
    - Busca reminders onde scheduledAt <= now E sent = false
    - Para cada um: envia mensagem para ADMIN\_PHONE via WhatsApp
    - Marca sent = true
 3\. O aiAssistant.js já deve conseguir criar reminders via CREATE\_REMINDER intent.
    Faça o parser da data/hora em português: 'amanhã às 10h', 'sexta às 15h30', etc.
    Use date-fns para o parsing.
|
**💡  Esses prompts da Fase IA foram desenhados para serem adicionados sem quebrar o MVP. Cada um é incremental — você pode implementar um de cada vez quando quiser.**
```text

```
Itaflix — Guia Vibe Coding  •  Complemento do MVP v2.0  •  Use com Cursor, Windsurf ou qualquer IDE com IA
Itaflix — Guia Vibe Coding  |  Página  de
