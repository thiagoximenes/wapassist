
**ITAFLIX**
Sistema de Gestão de Assinaturas IPTV
Guia de Implementação do MVP — v2.0
|**React + Vite**|**Node + Fastify**|**PostgreSQL**|**Evolution API**|
| :-: | :-: | :-: | :-: |
Dashboard web  •  Pagamento Pix automático  •  Notificações WhatsApp  •  ~R$ 30/mês

# **Visão Geral do Sistema**
O Itaflix MVP é uma aplicação web privada onde você gerencia todos os seus clientes de IPTV: cadastro, planos, vencimentos, pagamentos e notificações automáticas via WhatsApp. A integração com o Mercado Pago detecta pagamentos Pix automaticamente, sem nenhuma ação manual.

## **Stack Tecnológica**
|**Camada**|**Tecnologia**|**Função**|
| :- | :- | :- |
|Frontend (UI)|React + Vite + TailwindCSS|Dashboard de gerenciamento — deploy gratuito na Vercel|
|Backend (API)|Node.js + Fastify|Lógica de negócio, webhooks e agendamentos — deploy gratuito no Render|
|Banco de Dados|PostgreSQL (Neon.tech)|Armazenamento de clientes, planos, pagamentos — tier gratuito|
|Pagamentos|Mercado Pago Pix|Webhook automático detecta pagamento confirmado|
|WhatsApp|Evolution API (VPS)|Envia cobranças, confirmações e futuras notificações IA|
|Scheduler|node-cron (no backend)|Disparo automático D-1 antes do vencimento, sem custo extra|
|Autenticação|JWT simples|Login com senha para acessar a dashboard — só você acessa|

## **Custo Mensal do MVP**
|**Serviço**|**Plano**|**Custo**|
| :- | :- | :- |
|VPS Linux (Hostinger KVM 1 ou Hetzner CX11)|Básico|~R$ 20–25/mês|
|React (Vercel)|Free|R$ 0|
|Node.js API (Render)|Free tier|R$ 0|
|PostgreSQL (Neon.tech)|Free tier (0,5 GB)|R$ 0|
|Evolution API (WhatsApp)|Self-hosted na VPS|R$ 0|
|Mercado Pago (taxa Pix)|~0,99% por transação|~R$ 0,30 por cliente|
|**TOTAL (30 clientes)**||**~R$ 30/mês**|

## **Fluxo Completo do Sistema**
|**#**|**Evento**|**O que acontece**|
| :- | :- | :- |
|**1**|**Novo cliente cadastrado**|Você preenche nome, telefone, plano e data de vencimento inicial na dashboard. Sistema salva no banco.|
|**2**|**D-1 antes do vencimento (automático)**|node-cron roda à meia-noite, identifica clientes com vencimento amanhã e dispara cobrança via WhatsApp com link Pix gerado pelo Mercado Pago.|
|**3**|**Cliente paga o Pix**|Mercado Pago confirma o pagamento e envia webhook para o backend.|
|**4**|**Backend recebe webhook do MP**|Identifica o cliente pelo external\_reference do Pix, registra o pagamento e calcula nova data de vencimento.|
|**5**|**Cálculo da nova data (lógica de plano)**|Se pagou antes do vencimento: nova data = vencimento atual + dias do plano. Se pagou depois: nova data = data do pagamento + dias do plano.|
|**6**|**Confirmação automática no WhatsApp**|Cliente recebe mensagem: Pagamento confirmado! Nova validade: DD/MM/AAAA.|
|**7**|**Dashboard atualizada em tempo real**|Status do cliente muda para Ativo com nova data visível para você.|

## **Roadmap: MVP → IA**
|**Fase**|**Quando**|**O que inclui**|
| :- | :- | :- |
|**MVP (Fase 1)**|Agora — 2 a 3 semanas|Dashboard CRUD, pagamento Pix automático, notificações WhatsApp de cobrança e confirmação|
|**Fase 2**|Após o MVP estável|Relatórios financeiros, histórico de pagamentos por cliente, múltiplos links Pix por plano|
|**Fase 3 — IA**|Futuro (3 a 6 meses)|Comandos por texto/voz no WhatsApp: adicionar notas, criar lembretes, receber alertas de vencimento para você, consultar status de clientes com linguagem natural|
|**01**| **VPS + Evolution API (WhatsApp)**
 Ubuntu 22.04 + Docker + SSL — base de toda a comunicação
|
| :-: | :- |

## **1.1 — Contratar a VPS**
A VPS hospeda somente a Evolution API. O backend e o frontend ficam em serviços gratuitos (Render + Vercel).
|**Provedor**|**Plano**|**Custo**|**Diferencial**|
| :- | :- | :- | :- |
|Hostinger ⭐|KVM 1 (1 vCPU / 2 GB)|~R$ 20/mês|Suporte PT-BR, datacenter SP|
|Hetzner|CX11 (1 vCPU / 2 GB)|~R$ 18/mês|Melhor custo, datacenter EU|
Ao criar a VPS, selecione obrigatoriamente: Ubuntu 22.04 LTS, mínimo 2 GB RAM.

## **1.2 — Apontar DNS**
No painel do seu domínio itaflix.com.br, crie o registro:
|**Tipo**|**Nome**|**Valor**|**TTL**|
| :- | :- | :- | :- |
|A|api|IP\_DA\_SUA\_VPS|300|
**⚠️  Aguarde 10 minutos para propagar. Verifique em dnschecker.org antes de prosseguir para o SSL.**
```text

```

## **1.3 — Configurar o Servidor**
Conecte via SSH: ssh root@IP\_DA\_VPS e execute os comandos em ordem:

### **Atualizar sistema e instalar base**
apt update && apt upgrade -y
apt install -y curl git ufw nano

### **Firewall**
ufw allow OpenSSH && ufw allow 80/tcp && ufw allow 443/tcp && ufw enable

### **Docker**
curl -fsSL https://get.docker.com | sh
usermod -aG docker $USER && newgrp docker

### **Nginx + SSL**
apt install -y nginx certbot python3-certbot-nginx
systemctl enable nginx && systemctl start nginx

## **1.4 — Subir a Evolution API com Docker**
mkdir -p ~/evolution && cd ~/evolution && nano docker-compose.yml
| version: '3.8'
 services:
   evolution-api:
     image: atendai/evolution-api:latest
     container\_name: evolution\_api
     restart: always
     ports:
       - "8080:8080"
     environment:
       AUTHENTICATION\_TYPE: apikey
       AUTHENTICATION\_API\_KEY: "GERE\_COM\_openssl\_rand\_-hex\_32"
       AUTHENTICATION\_EXPOSE\_IN\_FETCH\_INSTANCES: "true"
       DEL\_INSTANCE: "false"
       DATABASE\_ENABLED: "false"
     volumes:
       - evolution\_instances:/evolution/instances
       - evolution\_store:/evolution/store
 volumes:
   evolution\_instances:
   evolution\_store:
|
| :- |
docker compose up -d
docker logs evolution\_api --tail 20
**✅  O log deve exibir: Server is listening on port 8080**
```text

```

## **1.5 — Configurar Nginx e SSL**
nano /etc/nginx/sites-available/evolution
| server {
     listen 80;
     server\_name api.itaflix.com.br;
     location / {
         proxy\_pass http://localhost:8080;
         proxy\_http\_version 1.1;
         proxy\_set\_header Upgrade $http\_upgrade;
         proxy\_set\_header Connection 'upgrade';
         proxy\_set\_header Host $host;
         proxy\_set\_header X-Real-IP $remote\_addr;
         proxy\_set\_header X-Forwarded-Proto $scheme;
         proxy\_read\_timeout 120s;
     }
 }
|
| :- |
ln -s /etc/nginx/sites-available/evolution /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
certbot --nginx -d api.itaflix.com.br --email SEU@EMAIL.COM --agree-tos --non-interactive

## **1.6 — Conectar WhatsApp (QR Code)**
Execute no terminal (substitua SUA\_CHAVE pela chave do docker-compose):
curl -X POST https://api.itaflix.com.br/instance/create -H "Content-Type: application/json" -H "apikey: SUA\_CHAVE" -d '{"instanceName": "itaflix", "qrcode": true}'
curl -X GET "https://api.itaflix.com.br/instance/connect/itaflix" -H "apikey: SUA\_CHAVE"
A resposta contém um campo base64 com a imagem do QR Code. Acesse base64.guru/converter/decode/image, cole o valor e escaneie com o WhatsApp do número que enviará as mensagens.
**⚠️  Use um número dedicado para o Itaflix, não o seu pessoal. Após escanear, o status deve aparecer como open.**
```text

```
|**02**| **Banco de Dados — PostgreSQL (Neon.tech)**
 Gratuito, gerenciado, sem necessidade de VPS
|
| :-: | :- |

## **2.1 — Criar a Conta e o Banco**
1. Acesse neon.tech e crie uma conta gratuita (pode usar o login Google)
1. Clique em New Project e dê o nome itaflix
1. Selecione a região mais próxima: AWS São Paulo (sa-east-1)
1. Após criar, clique em Connection Details e copie a Connection String — ela tem este formato:
postgresql://usuario:senha@host.neon.tech/itaflix?sslmode=require
**ℹ️  Salve essa string em local seguro. Ela será usada como variável de ambiente DATABASE\_URL no backend.**
```text

```

## **2.2 — Estrutura do Banco (Tabelas)**
O backend cria as tabelas automaticamente na primeira execução via migrations. A estrutura é:
| -- Clientes
 CREATE TABLE clients (
   id          SERIAL PRIMARY KEY,
   name        VARCHAR(100) NOT NULL,
   phone       VARCHAR(20)  NOT NULL UNIQUE,  -- formato: 5521999998888
   email       VARCHAR(100),
   plan        VARCHAR(20)  NOT NULL,          -- monthly | quarterly | semiannual | annual
   status      VARCHAR(20)  DEFAULT 'active',  -- active | inactive | overdue
   due\_date    DATE         NOT NULL,           -- próximo vencimento
   created\_at  TIMESTAMP    DEFAULT NOW()
 );
 -- Pagamentos
 CREATE TABLE payments (
   id               SERIAL PRIMARY KEY,
   client\_id        INT REFERENCES clients(id),
   mp\_payment\_id    VARCHAR(100),               -- ID do Mercado Pago
   amount           DECIMAL(10,2),
   paid\_at          TIMESTAMP,
   new\_due\_date     DATE,
   created\_at       TIMESTAMP DEFAULT NOW()
 );
 -- Notas (para fase IA: anotações por voz/texto)
 CREATE TABLE notes (
   id          SERIAL PRIMARY KEY,
   client\_id   INT REFERENCES clients(id),
   content     TEXT         NOT NULL,
   created\_at  TIMESTAMP    DEFAULT NOW()
 );
|
| :- |
|**03**| **Backend — Node.js + Fastify (Render)**
 API REST, lógica de negócio, webhooks e scheduler
|
| :-: | :- |

## **3.1 — Estrutura do Projeto**
| itaflix-api/
 ├── src/
 │   ├── server.js          # entry point
 │   ├── db.js              # conexão PostgreSQL
 │   ├── routes/
 │   │   ├── clients.js     # CRUD de clientes
 │   │   ├── payments.js    # registro e webhook MP
 │   │   └── auth.js        # login JWT
 │   ├── services/
 │   │   ├── whatsapp.js    # integração Evolution API
 │   │   ├── mercadopago.js # geração de Pix e webhook
 │   │   └── scheduler.js   # node-cron: cobranças D-1
 │   └── middleware/
 │       └── auth.js        # verificar JWT nas rotas
 ├── .env                   # variáveis de ambiente
 └── package.json
|
| :- |

## **3.2 — Variáveis de Ambiente (.env)**
| # Banco de dados
 DATABASE\_URL=postgresql://usuario:senha@host.neon.tech/itaflix?sslmode=require
 # Autenticação
 JWT\_SECRET=gere\_com\_openssl\_rand\_-hex\_32
 ADMIN\_PASSWORD=sua\_senha\_forte\_aqui
 # Mercado Pago
 MP\_ACCESS\_TOKEN=seu\_access\_token\_de\_producao
 MP\_WEBHOOK\_SECRET=chave\_para\_validar\_webhooks\_mp
 # Evolution API (WhatsApp)
 EVOLUTION\_URL=https://api.itaflix.com.br
 EVOLUTION\_APIKEY=sua\_chave\_evolution
 EVOLUTION\_INSTANCE=itaflix
 # App
 PORT=3000
 FRONTEND\_URL=https://itaflix.vercel.app
|
| :- |

## **3.3 — Lógica de Cálculo da Nova Data de Vencimento**
Este é o núcleo do sistema. O código abaixo implementa a regra de negócio completa:
| // services/billing.js
 const PLAN\_DAYS = {
   monthly:     30,
   quarterly:   90,
   semiannual: 180,
   annual:     365,
 };
 function calculateNewDueDate(currentDueDate, paidAt, plan) {
   const due  = new Date(currentDueDate);
   const paid = new Date(paidAt);
   const days = PLAN\_DAYS[plan];
   // Se pagou ANTES ou NO DIA do vencimento: mantém a sequência
   if (paid <= due) {
     const next = new Date(due);
     next.setDate(next.getDate() + days);
     return next;
   }
   // Se pagou DEPOIS do vencimento: reinicia a partir do pagamento
   const next = new Date(paid);
   next.setDate(next.getDate() + days);
   return next;
 }
|
| :- |

## **3.4 — Webhook do Mercado Pago**
Quando o cliente paga o Pix, o Mercado Pago chama este endpoint automaticamente:
| // routes/payments.js (simplificado)
 fastify.post('/webhook/mercadopago', async (req, reply) => {
   const { type, data } = req.body;
   if (type !== 'payment') return reply.send({ ok: true });
   const payment = await mercadopago.getPayment(data.id);
   if (payment.status !== 'approved') return reply.send({ ok: true });
   // external\_reference = phone do cliente (definido ao gerar o Pix)
   const client = await db.getClientByPhone(payment.external\_reference);
   if (!client) return reply.status(404).send({ error: 'cliente nao encontrado' });
   const newDue = calculateNewDueDate(client.due\_date, new Date(), client.plan);
   await db.updateClientDueDate(client.id, newDue);
   await db.createPayment({ client\_id: client.id, amount: payment.transaction\_amount,
                            mp\_payment\_id: data.id, paid\_at: new Date(), new\_due\_date: newDue });
   // Envia confirmação no WhatsApp
   await whatsapp.sendConfirmation(client.phone, client.name, newDue);
   return reply.send({ ok: true });
 });
|
| :- |

## **3.5 — Scheduler: Cobrança Automática D-1**
| // services/scheduler.js
 import cron from 'node-cron';
 // Roda todo dia às 09:00
 cron.schedule('0 9 \* \* \*', async () => {
   const tomorrow = new Date();
   tomorrow.setDate(tomorrow.getDate() + 1);
   const clients = await db.getClientsDueOn(tomorrow);
   for (const client of clients) {
     const pixLink = await mercadopago.createPixLink({
       title:              `Itaflix - Renovacao ${client.plan}`,
       amount:             PLAN\_PRICES[client.plan],
       external\_reference: client.phone,
       expiration\_hours:   48,
     });
     await whatsapp.sendBillingReminder(client.phone, client.name, tomorrow, pixLink);
   }
 });
|
| :- |

## **3.6 — Deploy no Render (Gratuito)**
1. Crie uma conta em render.com
1. Clique em New > Web Service e conecte ao repositório GitHub do projeto
1. Configure:
   - Build Command: npm install
   - Start Command: node src/server.js
   - Environment: Node
1. Adicione todas as variáveis do .env no painel de Environment Variables do Render
1. Após o deploy, copie a URL pública (ex: https://itaflix-api.onrender.com)
1. Cadastre a URL do webhook no Mercado Pago: https://itaflix-api.onrender.com/webhook/mercadopago
**⚠️  O plano gratuito do Render hiberna após 15 min sem uso. Para produção, considere o plano pago (US$ 7/mês) ou mantenha a API sempre ativa com um ping periódico.**
```text

```
|**04**| **Frontend — Dashboard React (Vercel)**
 Interface de gerenciamento dos clientes
|
| :-: | :- |

## **4.1 — Telas da Dashboard**
|**Tela**|**Funcionalidades**|
| :- | :- |
|Login|Autenticação com senha (JWT). Só você acessa a dashboard.|
|Visão Geral|Cards de resumo: total de clientes, quantos vencem esta semana, receita do mês. Lista dos clientes com vencimento próximo em destaque.|
|Clientes|Tabela com todos os clientes. Filtros por status (ativo, vencido, inativo) e plano. Botões de ação: editar, ver histórico, marcar inativo.|
|Novo Cliente|Formulário: nome, telefone, e-mail, plano (mensal/trimestral/semestral/anual), data de vencimento inicial.|
|Detalhe do Cliente|Dados completos + histórico de pagamentos + campo de notas. Botão para reenviar cobrança manualmente.|
|Pagamentos|Histórico geral de todos os pagamentos com filtro por período.|

## **4.2 — Criar o Projeto React**
npm create vite@latest itaflix-dashboard -- --template react
cd itaflix-dashboard
npm install axios react-router-dom @tanstack/react-query date-fns
npm install -D tailwindcss postcss autoprefixer && npx tailwindcss init -p

## **4.3 — Variável de Ambiente do Frontend**
Crie o arquivo .env.local na raiz do projeto frontend:
VITE\_API\_URL=https://itaflix-api.onrender.com

## **4.4 — Deploy na Vercel**
1. Acesse vercel.com e crie uma conta gratuita
1. Clique em Add New > Project e importe o repositório GitHub do frontend
1. Adicione a variável VITE\_API\_URL com a URL do backend no Render
1. Clique em Deploy — a Vercel detecta automaticamente que é um projeto Vite/React
1. Após o deploy, você terá uma URL como https://itaflix-dashboard.vercel.app
**ℹ️  Para domínio customizado (ex: admin.itaflix.com.br), vá em Settings > Domains na Vercel e adicione o subdomínio. É gratuito.**
```text

```
|**05**| **Integração Mercado Pago**
 Conta, credenciais e configuração de webhook
|
| :-: | :- |

## **5.1 — Criar Conta e Obter Credenciais**
1. Acesse mercadopago.com.br e crie uma conta de vendedor (gratuita)
1. No painel, vá em Seu negócio > Configurações > Credenciais
1. Copie o Access Token de PRODUÇÃO (começa com APP\_USR-)
1. Cole no campo MP\_ACCESS\_TOKEN do .env do backend

## **5.2 — Configurar o Webhook no Mercado Pago**
1. No painel MP, vá em Seu negócio > Configurações > Notificações (Webhooks)
1. Clique em Adicionar nova URL de webhook
1. URL: https://itaflix-api.onrender.com/webhook/mercadopago
1. Evento: Pagamentos
1. Salve e copie o secret gerado — ele vai para MP\_WEBHOOK\_SECRET no .env

## **5.3 — Lógica de Geração do Link Pix por Cliente**
Cada cobrança gera um Pix personalizado com o telefone do cliente como external\_reference, permitindo identificar automaticamente quem pagou:
| // services/mercadopago.js
 import MercadoPagoConfig, { Preference } from 'mercadopago';
 const client = new MercadoPagoConfig({ accessToken: process.env.MP\_ACCESS\_TOKEN });
 export async function createPixLink({ title, amount, external\_reference }) {
   const preference = new Preference(client);
   const result = await preference.create({
     body: {
       items: [{ title, quantity: 1, unit\_price: amount }],
       payment\_methods: { excluded\_payment\_types: [{ id: 'credit\_card' }, { id: 'debit\_card' }] },
       external\_reference,  // phone do cliente
       expiration\_date\_to: new Date(Date.now() + 48 \* 60 \* 60 \* 1000).toISOString(),
     }
   });
   return result.init\_point; // URL do checkout Pix
 }
|
| :- |
|**06**| **Mensagens WhatsApp**
 Templates de cobrança, confirmação e alertas para você
|
| :-: | :- |

## **6.1 — Template de Cobrança (D-1)**
| // Enviada automaticamente 1 dia antes do vencimento
 Olá, \*{NOME}\*! 👋
 Sua assinatura \*Itaflix\* vence amanhã, \*{DATA}\*.
 Para renovar, pague o Pix abaixo:
 💰 Valor: R$ {VALOR}
 🔗 Link: {LINK\_PIX}
 O link expira em 48 horas.
 Qualquer dúvida é só chamar! 😊
|
| :- |

## **6.2 — Template de Confirmação de Pagamento**
| // Enviada automaticamente após detecção do Pix no webhook
 ✅ \*Pagamento confirmado!\*
 Olá, \*{NOME}\*!
 Recebemos seu pagamento com sucesso.
 📅 Nova validade: \*{NOVA\_DATA}\*
 📦 Plano: {PLANO}
 Bom entretenimento! 🎬
 — Equipe Itaflix
|
| :- |

## **6.3 — Alerta para Você (Cliente em Atraso)**
| // Enviada para o SEU número quando um cliente passa 3 dias em atraso
 ⚠️ \*Alerta Itaflix — Cliente em atraso\*
 Cliente: {NOME}
 Telefone: {TELEFONE}
 Vencimento era: {DATA\_VENC}
 Dias em atraso: {DIAS}
 Considere entrar em contato manualmente.
|
| :- |
**ℹ️  Os alertas para você são enviados para um número diferente do cliente. Configure ADMIN\_PHONE no .env do backend com o seu número pessoal.**
```text

```
|**07**| **Fase Futura — IA por Voz e Texto**
 Como a integração inteligente será construída sobre o MVP
|
| :-: | :- |

## **7.1 — O Que Será Possível Fazer**
Com o MVP funcionando, a camada de IA permitirá que você interaja com o sistema diretamente pelo WhatsApp, como se estivesse conversando com um assistente:
|**Você diz / escreve**|**O sistema faz**|
| :- | :- |
|"Anota que o João pediu para mudar para plano trimestral"|Salva uma nota na ficha do cliente João no banco de dados|
|"Quais clientes vencem essa semana?"|Consulta o banco e responde com a lista formatada no WhatsApp|
|"Me lembra amanhã às 10h de ligar para a Maria"|Cria um agendamento; no horário certo você recebe a notificação|
|"João pagou, pode confirmar"|Registra o pagamento manualmente, calcula nova data e envia confirmação ao João|
|Mensagem de voz com qualquer comando acima|Transcreve com Whisper (OpenAI) e executa a mesma lógica|

## **7.2 — Arquitetura da Camada de IA**
|**Componente**|**Tecnologia e Função**|
| :- | :- |
|Entrada de texto|Webhook da Evolution API recebe mensagens enviadas ao número do Itaflix|
|Entrada de voz|Arquivo de áudio transcrito com Whisper API (OpenAI) — custo muito baixo, ~US$ 0,006/min|
|Interpretação|GPT-4o Mini (OpenAI) analisa a intenção e extrai entidades: nome do cliente, ação, data|
|Execução|Backend executa a ação correspondente: salvar nota, buscar clientes, criar agendamento|
|Resposta|Resultado enviado de volta para você via WhatsApp|
**ℹ️  Todo o custo da fase IA fica em torno de US$ 2 a 5/mês com o volume de uso pessoal do Itaflix. A base (MVP) não tem esse custo.**
```text

```
|**08**| **Checklist de Implementação**
 Do início ao MVP funcional
|
| :-: | :- |

## **Fase 1 — Infraestrutura**
||**Tarefa**|**Resultado esperado**|
| :- | :- | :- |
|☐|**VPS Ubuntu 22.04 contratada**|IP e acesso SSH em mãos|
|☐|**DNS api.itaflix.com.br apontado**|Propagação confirmada no dnschecker.org|
|☐|**Docker instalado na VPS**|docker --version retorna versão|
|☐|**Evolution API rodando**|Container ativo, porta 8080 respondendo|
|☐|**SSL ativo em api.itaflix.com.br**|HTTPS sem alertas de segurança|
|☐|**WhatsApp conectado (QR Code)**|Status da instância: open|
|☐|**Mensagem de teste enviada**|WhatsApp recebe a mensagem de teste|

## **Fase 2 — Backend e Banco**
||**Tarefa**|**Resultado esperado**|
| :- | :- | :- |
|☐|**Conta Neon.tech criada**|Banco itaflix criado, connection string copiada|
|☐|**Repositório backend criado no GitHub**|Estrutura de pastas conforme seção 3.1|
|☐|**.env configurado com todas as variáveis**|Nenhuma variável vazia|
|☐|**Tabelas criadas via migration**|Tabelas clients, payments e notes existem|
|☐|**Backend deployado no Render**|URL pública funcionando|
|☐|**Webhook MP registrado e testado**|MP envia notificação, backend processa|
|☐|**Scheduler funcionando**|Log mostra verificação diária às 09:00|

## **Fase 3 — Frontend e Integração**
||**Tarefa**|**Resultado esperado**|
| :- | :- | :- |
|☐|**Projeto React criado e configurado**|Vite + Tailwind funcionando localmente|
|☐|**Dashboard deployada na Vercel**|URL pública acessível|
|☐|**Login com JWT funcionando**|Senha protege o acesso|
|☐|**CRUD de clientes funcionando**|Criar, editar, listar e inativar clientes|
|☐|**Conta Mercado Pago configurada**|Access Token de produção ativo|
|☐|**Teste end-to-end realizado**|Pix pago → Tasks atualizado → WhatsApp enviado|
|☐|**Clientes existentes migrados**|30 clientes cadastrados com datas corretas|

# **Solução de Problemas Comuns**
|**Problema**|**Solução**|
| :- | :- |
|**Render hiberna e perde o webhook do MP**|Configure um serviço de uptime como uptimerobot.com para pingar a URL a cada 5 min (gratuito). Ou use o plano pago do Render (US$ 7/mês).|
|**Webhook do Mercado Pago não chega**|Confirme que a URL está acessível publicamente. Teste com: curl -X POST https://itaflix-api.onrender.com/webhook/mercadopago|
|**Cliente paga mas WhatsApp não envia**|Verifique o log do backend. A instância Evolution pode ter desconectado — reescaneie o QR Code.|
|**Instância WhatsApp desconecta**|Mantenha o celular com bateria e conexão estável. Não use o mesmo número no WhatsApp Web simultaneamente.|
|**Neon.tech: banco suspendido**|O free tier suspende após 7 dias sem atividade. O Render fazendo ping no banco evita isso.|
|**Data de vencimento calculada errada**|Verifique se o campo due\_date está sendo salvo como DATE sem fuso horário. Use always UTC no backend.|
|**MP retorna erro na geração do Pix**|Confirme que o Access Token é de PRODUÇÃO, não de sandbox. Tokens de teste começam com TEST-.|
**✅  Este guia cobre o MVP completo. Quando a infraestrutura estiver no ar, você receberá o código-fonte completo do backend e frontend para cada etapa. Avance uma etapa de cada vez.**
```text

```
Itaflix MVP v2.0  •  Stack: React + Fastify + PostgreSQL + Evolution API + Mercado Pago  •  ~R$ 30/mês
Itaflix — MVP de Automação  |  Página  de
