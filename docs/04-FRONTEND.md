# 04 — Frontend: Dashboard React

> **Objetivo:** Construir a dashboard de gerenciamento com 7 telas, design system dark e integração completa com a API.  
> **Stack:** React + Vite + TailwindCSS + React Query + Lucide React  
> **Hospedagem:** Vercel (free tier)  
> **Tempo estimado:** 6–10 horas de desenvolvimento

---

## Mapa de Telas

| # | Tela | Rota | Objetivo |
|---|---|---|---|
| 1 | **Login** | `/login` | Autenticação com senha. Única entrada para a dashboard. |
| 2 | **Visão Geral** | `/` | KPIs do negócio, alertas e agenda do dia. |
| 3 | **Clientes** | `/clientes` | Listagem, busca, filtros e ações rápidas. |
| 4 | **Novo/Editar Cliente** | `/clientes/novo` e `/clientes/:id/editar` | Formulário de cadastro e edição. |
| 5 | **Detalhe do Cliente** | `/clientes/:id` | Ficha completa: dados, pagamentos, notas. |
| 6 | **Pagamentos** | `/pagamentos` | Histórico financeiro com filtros por período. |
| 7 | **Calendário** | `/calendario` | Cobranças, tarefas e recorrências com notificações WhatsApp. |

---

## Etapa 4.1 — Criar e Configurar o Projeto

```bash
npm create vite@latest wapassist-dashboard -- --template react
cd wapassist-dashboard
npm install
npm install axios react-router-dom @tanstack/react-query date-fns
npm install lucide-react react-input-mask
npm install @fullcalendar/react @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Checklist

- [ ] Projeto criado com Vite + template React
- [ ] Todas as dependências instaladas
- [ ] TailwindCSS configurado com `postcss` e `autoprefixer`
- [ ] `tailwind.config.js` com `content: ['./index.html', './src/**/*.{js,jsx}']`

---

## Etapa 4.2 — Design System

### Paleta de Cores (`src/styles/tokens.css`)

Tema **dark de alta densidade** com ciano elétrico como cor de destaque.

```css
:root {
  /* Backgrounds */
  --bg-base:    #080C14;  /* fundo principal da página */
  --bg-surface: #0F1825;  /* cards, dropdowns, modais */
  --bg-panel:   #151E2D;  /* sidebar, topbar */
  --bg-hover:   #1A2438;  /* hover em itens de lista */
  --bg-active:  #1E2D45;  /* item de menu ativo */

  /* Bordas */
  --border:     #1E2D42;
  --border-md:  #253347;

  /* Texto */
  --text-primary:   #E2E8F0;
  --text-secondary: #94A3B8;
  --text-muted:     #4B6280;

  /* Accent — Ciano */
  --cyan-400: #22D3EE;
  --cyan-500: #06B6D4;  /* principal */
  --cyan-600: #0891B2;
  --cyan-900: #164E63;
  --cyan-950: #0C3040;

  /* Status */
  --green:  #10B981;  --green-bg:  #022C22;
  --yellow: #F59E0B;  --yellow-bg: #2D1B00;
  --red:    #EF4444;  --red-bg:    #2D0707;
  --orange: #F97316;  --orange-bg: #2A1200;

  /* Tipografia */
  --font-mono: 'DM Mono', monospace;
  --font-sans: 'DM Sans', sans-serif;

  /* Sombras */
  --shadow-card: 0 4px 24px rgba(0,0,0,0.4);
  --shadow-glow: 0 0 20px rgba(6,182,212,0.15);
}
```

### Fontes (`index.html`)

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
```

### Global CSS (`src/index.css`)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

* { font-family: var(--font-sans); }
body { background: var(--bg-base); color: var(--text-primary); }
code, pre, .mono { font-family: var(--font-mono); }
```

---

## Etapa 4.3 — Componentes Base (`src/components/ui/`)

Criar estes componentes **antes de qualquer tela**. São usados em todas as páginas.

| Componente | Arquivo | Descrição |
|---|---|---|
| `Badge` | `ui/Badge.jsx` | Pill colorido para status e planos |
| `Button` | `ui/Button.jsx` | Variantes: `primary`, `ghost`, `danger`. Tamanhos: `sm`, `md`, `lg` |
| `Card` | `ui/Card.jsx` | Container com fundo surface, borda e sombra padrão |
| `Input` | `ui/Input.jsx` | Input com label, ícone opcional e estado de erro |
| `Select` | `ui/Select.jsx` | Dropdown estilizado |
| `Modal` | `ui/Modal.jsx` | Overlay com backdrop blur e animação |
| `Toast` | `ui/Toast.jsx` | Notificação temporária (sucesso, erro, info) |
| `Skeleton` | `ui/Skeleton.jsx` | Placeholder animado para loading |
| `Avatar` | `ui/Avatar.jsx` | Círculo com inicial do nome, cor gerada pelo nome |
| `StatCard` | `ui/StatCard.jsx` | Card de KPI: ícone, valor, label e variação |
| `EmptyState` | `ui/EmptyState.jsx` | Tela vazia com ícone, título e botão de ação |
| `Tooltip` | `ui/Tooltip.jsx` | Tooltip ao hover |

### Variantes do Badge

```javascript
const VARIANTS = {
  // Status
  active:    { bg: 'var(--green-bg)',   color: 'var(--green)',    label: 'Ativo' },
  overdue:   { bg: 'var(--red-bg)',     color: 'var(--red)',      label: 'Em atraso' },
  inactive:  { bg: '#1E293B',           color: '#64748B',         label: 'Inativo' },
  // Planos
  monthly:   { bg: 'var(--cyan-950)',   color: 'var(--cyan-400)', label: 'Mensal' },
  quarterly: { bg: '#1A1035',           color: '#A78BFA',         label: 'Trimestral' },
  semiannual:{ bg: '#1A2200',           color: '#84CC16',         label: 'Semestral' },
  annual:    { bg: 'var(--yellow-bg)',  color: 'var(--yellow)',   label: 'Anual' },
};
```

---

## Etapa 4.4 — Infraestrutura da Aplicação

### Cliente HTTP (`src/lib/api.js`)

```javascript
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

// Injeta JWT em todas as requisições
api.interceptors.request.use(config => {
  const token = localStorage.getItem('wapassist_token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Redireciona para login em caso de 401
api.interceptors.response.use(
  res => res,
  err => {
    if (err.response?.status === 401) {
      localStorage.removeItem('wapassist_token');
      window.location.href = '/login';
    }
    return Promise.reject(err);
  }
);
```

### React Query Client (`src/lib/queryClient.js`)

```javascript
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,
      retry: 1,
    },
  },
});
```

### Rotas e App (`src/App.jsx`)

```
Rotas públicas:
  /login → LoginPage

Rotas protegidas (dentro do Layout):
  /             → Visão Geral (HomePage)
  /clientes     → ClientsPage
  /clientes/novo → NewClientPage
  /clientes/:id  → ClientDetailPage
  /clientes/:id/editar → NewClientPage (modo edição)
  /pagamentos   → PaymentsPage
  /calendario   → CalendarPage

PrivateRoute: se não tiver JWT no localStorage → redireciona para /login
```

---

## Etapa 4.5 — Layout Global (`src/components/Layout.jsx`)

```
┌─────────────────────────────────────────────────────────────────────────┐
│  SIDEBAR (240px fixo)           │  MAIN AREA (flex-1)                  │
│  ──────────────────────────     │  ──────────────────────────────────  │
│                                 │  TOPBAR (60px)                       │
│  ◈  WAPASSIST          [logo]     │  [Título da página]   [avatar] [⚙️]  │
│  ─────────────────────────      │  ────────────────────────────────    │
│                                 │                                       │
│  ≡  Visão Geral                 │  CONTEÚDO DA PÁGINA                  │
│  ≡  Clientes         [badge N]  │  (cada tela renderiza aqui)          │
│  ≡  Pagamentos                  │                                       │
│  ≡  Calendário                  │                                       │
│                                 │                                       │
│  ─────────────────────────      │                                       │
│  STATUS WHATSAPP                │                                       │
│  ● Conectado / Desconectado     │                                       │
│                                 │                                       │
│  [Sair]                         │                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

**Comportamento do status WhatsApp:** Consulta `GET /api/whatsapp/status` a cada 30 segundos via React Query com `refetchInterval: 30000`.

---

## Etapa 4.6 — Tela 1: Login (`/login`)

```
┌────────────────────────────────────────────────────────────────┐
│              background: --bg-base (escuro)                    │
│              efeito: grid pattern sutil em ciano               │
│                                                                │
│                  ┌──────────────────────┐                     │
│                  │   ◈  WAPASSIST          │                    │
│                  │   Acesse sua conta    │                    │
│                  │                       │                    │
│                  │   [🔒 Senha ·········] │                   │
│                  │                       │                    │
│                  │   [   Entrar →  ]     │                    │
│                  │                       │                    │
│                  │   ● erro (se houver)  │                    │
│                  └──────────────────────┘                     │
└────────────────────────────────────────────────────────────────┘
```

**Comportamentos:**
- Toggle mostrar/ocultar senha no ícone à direita
- Spinner no botão durante a requisição
- Badge vermelho animado com mensagem de erro
- Enter no input envia o formulário
- Ao sucesso: salva JWT no `localStorage` e redireciona para `/`

**API:** `POST /api/auth/login` com `{ password }`

---

## Etapa 4.7 — Tela 2: Visão Geral (`/`)

```
┌─ TOPBAR ──────────────────────────────────────────────────────────────┐
│  Visão Geral   Quinta, 19 de Fevereiro                  [avatar] [⚙]  │
├───────────────────────────────────────────────────────────────────────┤
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │ CLIENTES │ │ VENCEM   │ │ EM ATRASO│ │ RECEITA  │  ← StatCards   │
│  │  Ativos  │ │ 7 dias   │ │          │ │  /mês    │                │
│  │    28    │ │    5     │ │    3     │ │ R$840    │                │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                │
│                                                                        │
│  ┌─ AGENDA DO DIA ──────────────┐  ┌─ ALERTAS ────────────────────┐  │
│  │ Cobranças enviadas hoje: 3   │  │ 🔴 Carlos — 5 dias atraso    │  │
│  │  ● João Silva   [✓ enviado]  │  │ 🟡 Ana — vence amanhã        │  │
│  │  ● Maria Santos [✓ enviado]  │  │ 🟡 Pedro — vence amanhã      │  │
│  └──────────────────────────────┘  └──────────────────────────────┘  │
│                                                                        │
│  ┌─ ATIVIDADE RECENTE (últimos pagamentos) ───────────────────────┐   │
│  │ [J] João Silva    Mensal    R$30   pago há 2h  ✓ Ativo         │   │
│  └────────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────────┘
```

**KPIs (StatCards):**

| Card | Ícone | Query | Cor |
|---|---|---|---|
| Clientes Ativos | `Users` | `COUNT(status = ACTIVE)` | `--cyan-500` |
| Vencem em 7 dias | `CalendarClock` | `COUNT(dueDate BETWEEN hoje e +7d)` | `--yellow` |
| Em Atraso | `AlertTriangle` | `COUNT(status = OVERDUE)` | `--red` |
| Receita do Mês | `TrendingUp` | `SUM(amount) WHERE paidAt >= início do mês` | `--green` |

**API:** `GET /api/dashboard/summary`

---

## Etapa 4.8 — Tela 3: Clientes (`/clientes`)

```
┌─ TOPBAR ─────────────────────────────────────────── [+ Novo Cliente] ─┐
│  Clientes   28 registros                                               │
├────────────────────────────────────────────────────────────────────────┤
│  [🔍 Buscar por nome ou telefone...] [Status ▾] [Plano ▾] [Exportar] │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  CLIENTE          TELEFONE      PLANO    VENCIMENTO  STATUS AÇÃO │  │
│  │  [◉] João Silva   21 9 9999-0000 Mensal  15/03 ✓    ● Ativo  ⋮  │  │
│  │  [◉] Carlos Lima  21 9 7777-0000 Trim.   10/02 !!   ● Atraso ⋮  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│  ← 1  2  3 →    Mostrando 1-15 de 28                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

**Lógica de cores na coluna Vencimento:**

| Condição | Cor / Ícone |
|---|---|
| Vence em > 7 dias | Texto muted normal |
| Vence em 4–7 dias | Amarelo claro |
| Vence em 1–3 dias | Amarelo intenso |
| Vence amanhã | Laranja + badge "AMANHÃ" |
| Vencido (atrasado) | Vermelho + dias em atraso |

**Menu de ações (⋮):**
- 👁 Ver detalhes → `/clientes/:id`
- ✏ Editar → `/clientes/:id/editar`
- 📲 Enviar cobrança → `POST /api/clients/:id/send-billing`
- 🚫 Desativar → `PUT /api/clients/:id` com `status: INACTIVE`

**Paginação:** 15 registros por página.  
**Exportar CSV:** Gera e baixa `.csv` com os dados filtrados.

**API:** `GET /api/clients?status=&plan=&search=`

---

## Etapa 4.9 — Tela 4: Novo/Editar Cliente

```
┌─ TOPBAR ───────────────────────────────────────────────────────────────┐
│  ← Clientes  /  Novo cliente                                           │
├────────────────────────────────────────────────────────────────────────┤
│  ┌─ DADOS PESSOAIS ─────────────────┐  ┌─ PLANO E VENCIMENTO ───────┐  │
│  │  [Nome completo             ]    │  │  Plano (radio cards):      │  │
│  │  [Telefone (99) 99999-9999  ]    │  │  ┌──────┐ ┌──────┐        │  │
│  │  [E-mail (opcional)         ]    │  │  │Mensal│ │Trim. │        │  │
│  └──────────────────────────────────┘  │  │ R$30 │ │ R$80 │        │  │
│                                        │  └──────┘ └──────┘        │  │
│                                        │  Data de vencimento:       │  │
│                                        │  [📅 DD/MM/AAAA    ]      │  │
│                                        │  ┌─ RESUMO ─────────────┐  │  │
│                                        │  │ Mensal • R$ 30       │  │  │
│                                        │  │ Vence em 30 dias     │  │  │
│                                        │  └──────────────────────┘  │  │
│                                        └────────────────────────────┘  │
│  [Cancelar]                                       [Salvar cliente →]   │
└──────────────────────────────────────────────────────────────────────────┘
```

**Validações:**

| Campo | Regra | Mensagem de erro |
|---|---|---|
| Nome | Obrigatório, mínimo 3 chars | "Nome precisa ter ao menos 3 caracteres" |
| Telefone | 11 dígitos após limpar máscara | "Telefone inválido — use (DDD) + 9 dígitos" |
| Telefone | Único no sistema | "Esse telefone já está cadastrado" |
| Plano | Um dos 4 planos válidos | "Selecione um plano" |
| Vencimento | Obrigatório, não pode ser passado (novo) | "Data de vencimento inválida" |

**Modo edição:** Se a rota contiver `:id`, carrega dados via `GET /api/clients/:id` e preenche o formulário. Título muda para "Editar cliente".

**APIs:**
- Novo: `POST /api/clients`
- Editar: `PUT /api/clients/:id`

---

## Etapa 4.10 — Tela 5: Detalhe do Cliente (`/clientes/:id`)

```
┌─ TOPBAR ──────────────────────────────────────────────────────────────┐
│  ← Clientes  /  João Silva                [Editar]  [Enviar cobrança]│
├───────────────────────────────────────────────────────────────────────┤
│  ┌─ CARD DO CLIENTE ──────────────────────────────────────────────┐  │
│  │  [  J  ]  João Silva                   ● Ativo   🔵 Mensal    │  │
│  │           📱 (21) 9 9999-0000                                  │  │
│  │           ✉  joao@email.com                                    │  │
│  │           📅 Vence em: 15/03/2025 (em 24 dias)                 │  │
│  │           📆 Cliente desde: 10/01/2024                         │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ┌─ ABAS ─────────────────────────────────────────────────────────┐  │
│  │  [ Histórico de Pagamentos ]  [ Notas ]                        │  │
│  ├────────────────────────────────────────────────────────────────┤  │
│  │  ABA HISTÓRICO:                                                │  │
│  │  DATA         VALOR   NOVA VALIDADE    STATUS                  │  │
│  │  13/02/2025  R$30    15/03/2025       ✓ Confirmado            │  │
│  │                                                                │  │
│  │  ABA NOTAS:                                                    │  │
│  │  [Escreva uma nota...                        ] [Salvar]        │  │
│  │  10/02 — 'Cliente pediu para mudar para plano trimestral'      │  │
│  └────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

**APIs:**
- `GET /api/clients/:id` (inclui `payments[]` e `notes[]`)
- `POST /api/clients/:id/send-billing`
- `POST /api/notes` com `{ clientId, content }`
- `DELETE /api/notes/:id`

---

## Etapa 4.11 — Tela 6: Pagamentos (`/pagamentos`)

```
┌─ TOPBAR ─────────────────────────────────────────────────────────────────┐
│  Pagamentos   R$ 840 recebido em Fevereiro                               │
├──────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│  │ Este mês     │  │ Mês anterior │  │ Total geral  │  ← mini KPIs     │
│  │ R$ 840       │  │ R$ 780       │  │ R$ 8.400     │                  │
│  └──────────────┘  └──────────────┘  └──────────────┘                  │
│                                                                           │
│  [🔍 Buscar cliente...]  [Período ▾]  [Plano ▾]                         │
│                                                                           │
│  DATA         CLIENTE        PLANO    VALOR   NOVA VALIDADE              │
│  13/02 14:32  João Silva     Mensal   R$30    15/03/2025                 │
│  ...                                                                      │
│  Total filtrado: R$ 840 (28 pagamentos)                                  │
└───────────────────────────────────────────────────────────────────────────┘
```

**Filtros de período:** Esta semana / Este mês / Mês anterior / 3 meses / Tudo

**API:** `GET /api/payments?clientId=&period=`

---

## Etapa 4.12 — Hook de Filtros Reutilizável (`src/hooks/useClientFilters.js`)

```javascript
import { useState, useMemo } from 'react';

export function useClientFilters(clients = []) {
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState('all');
  const [plan,   setPlan]   = useState('all');
  const [page,   setPage]   = useState(1);
  const PER_PAGE = 15;

  const filtered = useMemo(() => {
    return clients.filter(c => {
      const matchSearch = !search ||
        c.name.toLowerCase().includes(search.toLowerCase()) ||
        c.phone.includes(search.replace(/\D/g, ''));
      const matchStatus = status === 'all' || c.status.toLowerCase() === status;
      const matchPlan   = plan   === 'all' || c.plan.toLowerCase()   === plan;
      return matchSearch && matchStatus && matchPlan;
    });
  }, [clients, search, status, plan]);

  const paginated  = filtered.slice((page - 1) * PER_PAGE, page * PER_PAGE);
  const totalPages = Math.ceil(filtered.length / PER_PAGE);

  return {
    search, setSearch, status, setStatus, plan, setPlan,
    page, setPage, paginated, totalPages, total: filtered.length,
  };
}
```

---

## Etapa 4.13 — Deploy na Vercel

1. Acesse [vercel.com](https://vercel.com) e crie uma conta gratuita
2. Clique em **Add New > Project** e importe o repositório `wapassist-dashboard`
3. Framework Preset: **Vite** (detectado automaticamente)
4. Adicione a variável de ambiente:
   ```
   VITE_API_URL = https://wapassist-api.onrender.com
   ```
5. Clique em **Deploy**
6. (Opcional) Em **Settings > Domains**, adicione `admin.wapassist.com.br`

> 💡 Todo `git push` para `main` dispara redeploy automático na Vercel.

### Checklist

- [ ] Projeto React criado e configurado (Vite + Tailwind)
- [ ] Design system implementado (`tokens.css` + fontes)
- [ ] Todos os componentes `ui/` criados
- [ ] `src/lib/api.js` com interceptors de token e 401
- [ ] Sistema de rotas com `PrivateRoute` funcionando
- [ ] Layout com sidebar e status do WhatsApp
- [ ] Tela de Login funcionando com JWT
- [ ] Tela de Visão Geral com KPIs
- [ ] Tela de Clientes com filtros e paginação
- [ ] Formulário de Novo/Editar Cliente com validações
- [ ] Tela de Detalhe do Cliente com abas
- [ ] Tela de Pagamentos com filtros de período
- [ ] Dashboard deployada na Vercel com URL pública
- [ ] Login com JWT funcionando em produção
