
**ITAFLIX**
**Dashboard — Guia Completo de Criação**
Design System  •  Wireframes  •  Componentes  •  Código  •  Prompts
|**7 Telas**|**20+ Componentes**|**Design System**|**Código Pronto**|**Prompts IA**|
| :-: | :-: | :-: | :-: | :-: |

# **Mapa da Aplicação**
A dashboard do Itaflix tem 7 telas principais. Cada uma está documentada neste guia com wireframe, lista de componentes, lógica de estado e código-fonte completo.
|**#**|**Tela**|**Rota**|**Objetivo**|
| :- | :- | :- | :- |
|1|Login|/login|Autenticação segura. Única entrada para a dashboard.|
|2|Visão Geral (Home)|/|Panorama de saúde do negócio: KPIs, alertas e agenda do dia.|
|3|Clientes|/clientes|Listagem, busca, filtros e ações rápidas em todos os clientes.|
|4|Novo/Editar Cliente|/clientes/novo|Formulário completo de cadastro e edição de cliente.|
|5|Detalhe do Cliente|/clientes/:id|Ficha completa: dados, histórico de pagamentos, notas.|
|6|Pagamentos|/pagamentos|Histórico financeiro completo com filtros por período.|
|7|Calendário|/calendario|Visão mensal/semanal de cobranças, tarefas únicas e recorrentes com notificações WhatsApp.|
|**01**| **Design System**
 Tokens de cor, tipografia, componentes base e padrões visuais
|
| :-: | :- |

## **1.1 — Identidade Visual e Tema**
A dashboard usa um tema dark de alta densidade, com ciano elétrico como cor de destaque. O visual é inspirado em ferramentas de monitoramento profissional — limpo, técnico e de fácil leitura em telas por longos períodos.
**🎨  Fonte principal: DM Mono para dados e código, DM Sans para textos e labels. Importe do Google Fonts no index.html.**
```html

```

### **Paleta de Cores — CSS Variables**
**📄 src/styles/tokens.css**
```css

```
| :root {
   /\* Backgrounds \*/
   --bg-base:    #080C14;  /\* fundo principal da página \*/
   --bg-surface: #0F1825;  /\* cards, dropdowns, modais \*/
   --bg-panel:   #151E2D;  /\* sidebar, topbar \*/
   --bg-hover:   #1A2438;  /\* hover em itens de lista \*/
   --bg-active:  #1E2D45;  /\* item de menu ativo \*/
   /\* Bordas \*/
   --border:     #1E2D42;
   --border-md:  #253347;
   /\* Texto \*/
   --text-primary:   #E2E8F0;
   --text-secondary: #94A3B8;
   --text-muted:     #4B6280;
   /\* Accent — Ciano \*/
   --cyan-400: #22D3EE;
   --cyan-500: #06B6D4;  /\* principal \*/
   --cyan-600: #0891B2;
   --cyan-900: #164E63;
   --cyan-950: #0C3040;
   /\* Status \*/
   --green:  #10B981;  --green-bg:  #022C22;
   --yellow: #F59E0B;  --yellow-bg: #2D1B00;
   --red:    #EF4444;  --red-bg:    #2D0707;
   --orange: #F97316;  --orange-bg: #2A1200;
   /\* Tipografia \*/
   --font-mono: 'DM Mono', monospace;
   --font-sans: 'DM Sans', sans-serif;
   /\* Sombras \*/
   --shadow-card: 0 4px 24px rgba(0,0,0,0.4);
   --shadow-glow: 0 0 20px rgba(6,182,212,0.15);
 }
|

### **Importar Fontes**
**📄 index.html (dentro de <head>)**
```html

```
| <link rel="preconnect" href="https://fonts.googleapis.com">
 <link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
|
**📄 src/index.css (global)**
```css

```
| \* { font-family: var(--font-sans); }
 body { background: var(--bg-base); color: var(--text-primary); }
 code, pre, .mono { font-family: var(--font-mono); }
|

## **1.2 — Componentes Base**
Crie estes componentes em src/components/ui/ antes de começar as telas. Eles são usados em todas as páginas.
|**Componente**|**Arquivo**|**Descrição**|
| :- | :- | :- |
|Badge|ui/Badge.jsx|Pill colorido para status: Ativo, Em atraso, Inativo, Mensal, etc.|
|Button|ui/Button.jsx|Botão com variantes: primary, ghost, danger. Tamanhos: sm, md, lg.|
|Card|ui/Card.jsx|Container com fundo surface, borda sutil e sombra padrão.|
|Input|ui/Input.jsx|Input com label flutuante, ícone opcional e estado de erro.|
|Select|ui/Select.jsx|Dropdown estilizado com as opções do sistema (planos, status).|
|Modal|ui/Modal.jsx|Overlay com backdrop blur, animação de entrada e saída.|
|Toast|ui/Toast.jsx|Notificação temporária (sucesso, erro, info) no canto da tela.|
|Skeleton|ui/Skeleton.jsx|Placeholder animado para loading de tabelas e cards.|
|Avatar|ui/Avatar.jsx|Círculo com inicial do nome do cliente, cor gerada pelo nome.|
|StatCard|ui/StatCard.jsx|Card de KPI: ícone, valor numérico grande, label e variação.|
|EmptyState|ui/EmptyState.jsx|Tela vazia com ícone, título e botão de ação (ex: primeiro cliente).|
|Tooltip|ui/Tooltip.jsx|Tooltip simples que aparece ao hover em elementos de ação.|

### **Badge — Código completo**
**📄 src/components/ui/Badge.jsx**
```jsx

```
| const VARIANTS = {
   active:   { bg: 'var(--green-bg)',  color: 'var(--green)',  label: 'Ativo' },
   overdue:  { bg: 'var(--red-bg)',    color: 'var(--red)',    label: 'Em atraso' },
   inactive: { bg: '#1E293B',          color: '#64748B',       label: 'Inativo' },
   monthly:      { bg: 'var(--cyan-950)',  color: 'var(--cyan-400)', label: 'Mensal' },
   quarterly:    { bg: '#1A1035',          color: '#A78BFA',         label: 'Trimestral' },
   semiannual:   { bg: '#1A2200',          color: '#84CC16',         label: 'Semestral' },
   annual:       { bg: 'var(--yellow-bg)', color: 'var(--yellow)',   label: 'Anual' },
 };
 export function Badge({ variant, className = '' }) {
   const v = VARIANTS[variant] || VARIANTS.inactive;
   return (
     
       {v.label}
     
   );
 }
|

### **StatCard — Código completo**
**📄 src/components/ui/StatCard.jsx**
```jsx

```
| export function StatCard({ icon: Icon, label, value, sub, trend, color = 'var(--cyan-500)' }) {
   return (
     
       
         {label}
         
           <Icon size={16} style={{ color }} />
         
       
       
         
           {value}
         
         {trend && (
            0 ? 'var(--green)' : 'var(--red)' }}>
             {trend > 0 ? '▲' : '▼'} {Math.abs(trend)}%
           
         )}
       
       {sub &&  {sub}
}
     
   );
 }
|
|**02**| **Layout Global — Sidebar + Topbar**
 Estrutura que envolve todas as telas autenticadas
|
| :-: | :- |

## **2.1 — Wireframe do Layout**
**▸ WIREFRAME — Layout Global (todas as telas autenticadas)**
```text
┌─────────────────────────────────────────────────────────────────────────┐
│  SIDEBAR (240px fixo)           │  MAIN AREA (flex-1)                  │
│  ──────────────────────────     │  ──────────────────────────────────  │
│                                 │  TOPBAR (60px)                       │
│  ◈  ITAFLIX          [logo]     │  [Título da página]   [avatar] [⚙️]  │
│  ─────────────────────────      │  ────────────────────────────────    │
│                                 │                                       │
│  ≡  Visão Geral                 │  CONTEÚDO DA PÁGINA                  │
│  ≡  Clientes         [badge N]  │  (cada tela renderiza aqui)          │
│  ≡  Pagamentos                  │                                       │
│                                 │                                       │
│  ─────────────────────────      │                                       │
│                                 │                                       │
│  STATUS WHATSAPP                │                                       │
│  ● Conectado                    │                                       │
│                                 │                                       │
│  [Sair]                         │                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

## **2.2 — Código do Layout**
**📄 src/components/Layout.jsx**
```jsx

```
| import { NavLink, Outlet, useNavigate } from 'react-router-dom';
 import { LayoutDashboard, Users, CreditCard, LogOut, Wifi } from 'lucide-react';
 import { useQuery } from '@tanstack/react-query';
 import { api } from '../lib/api';
 const NAV = [
   { to: '/',           icon: LayoutDashboard, label: 'Visão Geral' },
   { to: '/clientes',   icon: Users,           label: 'Clientes'   },
   { to: '/pagamentos', icon: CreditCard,       label: 'Pagamentos' },
 ];
 export function Layout() {
   const navigate = useNavigate();
   const { data: status } = useQuery({
     queryKey: ['whatsapp-status'],
     queryFn: () => api.get('/whatsapp/status').then(r => r.data),
     refetchInterval: 30000,
   });
   const logout = () => { localStorage.removeItem('itaflix\_token'); navigate('/login'); };
   return (
     
       {/\* SIDEBAR \*/}
       <aside className='w-60 flex flex-col shrink-0'
         style={{ background: 'var(--bg-panel)', borderRight: '1px solid var(--border)' }}>
         {/\* Logo \*/}
         
           
             ITAFLIX
           
         
         {/\* Nav \*/}
         <nav className='flex-1 px-3 py-4 flex flex-col gap-1'>
           {NAV.map(({ to, icon: Icon, label }) => (
             <NavLink key={to} to={to} end={to === '/'}
               className={({ isActive }) => `flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all
                 ${isActive ? 'active-nav' : 'text-secondary hover:bg-hover'}`}>
               <Icon size={16} />
               {label}
             </NavLink>
           ))}
         </nav>
         {/\* Footer da sidebar \*/}
         
           
             
             
               WhatsApp {status?.connected ? 'conectado' : 'desconectado'}
             
           
           <button onClick={logout}
             className='flex items-center gap-2 px-3 py-2 rounded-lg text-sm w-full transition-colors'
             style={{ color: 'var(--text-muted)' }}>
             <LogOut size={14} /> Sair
           </button>
         
       </aside>
       {/\* MAIN \*/}
       <main className='flex-1 flex flex-col overflow-hidden'>
         <Outlet />
       </main>
     
   );
 }
|
**ℹ️  O componente <Outlet /> do React Router renderiza a tela atual dentro do layout. Cada página controla seu próprio header interno.**
```text

```
|**03**| **Tela 1 — Login**
 /login  •  Pública  •  Proteção JWT
|
| :-: | :- |

## **3.1 — Wireframe**
**▸ WIREFRAME — Login**
```text
┌────────────────────────────────────────────────────────────────┐
│              background: --bg-base (escuro)                    │
│                                                                │
│                  ┌──────────────────────┐                     │
│                  │   ◈  ITAFLIX          │  ← Card central    │
│                  │   ──────────────────  │                    │
│                  │   Acesse sua conta    │                    │
│                  │                       │                    │
│                  │   [🔒 Senha ·········] │  ← input          │
│                  │                       │                    │
│                  │   [   Entrar →  ]     │  ← botão primary   │
│                  │                       │                    │
│                  │   ● erro (se houver)  │                    │
│                  └──────────────────────┘                     │
│                                                                │
│         efeito de fundo: grid pattern sutil em ciano          │
└────────────────────────────────────────────────────────────────┘
```

## **3.2 — Detalhes de Design**
- Fundo: grade de linhas em ciano com opacidade 3% (background-image: SVG inline)
- Card: border-radius 16px, border 1px solid --border, backdrop-filter blur sutil
- Input de senha: toggle mostrar/ocultar no ícone à direita
- Botão: ao clicar, mostra spinner giratório enquanto faz a requisição
- Erro: badge vermelho animado deslizando de cima com mensagem 'Senha incorreta'
- Ao pressionar Enter no input, o formulário é enviado

## **3.3 — Código**
**📄 src/pages/LoginPage.jsx**
```jsx

```
| import { useState } from 'react';
 import { useNavigate } from 'react-router-dom';
 import { Eye, EyeOff, Lock, Loader2 } from 'lucide-react';
 import { api } from '../lib/api';
 export default function LoginPage() {
   const [pw, setPw]         = useState('');
   const [show, setShow]     = useState(false);
   const [loading, setLoading] = useState(false);
   const [error, setError]   = useState('');
   const navigate = useNavigate();
   async function handleSubmit(e) {
     e.preventDefault();
     setError(''); setLoading(true);
     try {
       const { data } = await api.post('/auth/login', { password: pw });
       localStorage.setItem('itaflix\_token', data.token);
       navigate('/');
     } catch {
       setError('Senha incorreta. Tente novamente.');
     } finally { setLoading(false); }
   }
   return (
     
       {/\* Grid background \*/}
       
       <form onSubmit={handleSubmit}
             className='relative z-10 w-full max-w-sm p-8 rounded-2xl flex flex-col gap-6'
             style={{ background: 'var(--bg-surface)', border: '1px solid var(--border)', boxShadow: '0 0 40px rgba(6,182,212,0.08)' }}>
         {/\* Header \*/}
         
           
             ITAFLIX
           
            Acesse sua conta
         
         {/\* Erro \*/}
         {error && (
           
             {error}
           
         )}
         {/\* Input \*/}
         
           <Lock size={15} className='absolute left-3 top-1/2 -translate-y-1/2' style={{ color: 'var(--text-muted)' }} />
           <input type={show ? 'text' : 'password'} value={pw} onChange={e => setPw(e.target.value)}
             placeholder='Senha de acesso'
             className='w-full pl-9 pr-10 py-3 rounded-lg text-sm outline-none transition-all'
             style={{ background: 'var(--bg-panel)', border: '1px solid var(--border)', color: 'var(--text-primary)' }} />
           <button type='button' onClick={() => setShow(!show)}
             className='absolute right-3 top-1/2 -translate-y-1/2' style={{ color: 'var(--text-muted)' }}>
             {show ? <EyeOff size={15} /> : <Eye size={15} />}
           </button>
         
         {/\* Botão \*/}
         <button type='submit' disabled={loading || !pw}
           className='flex items-center justify-center gap-2 py-3 rounded-lg text-sm font-semibold transition-all'
           style={{ background: loading ? 'var(--cyan-600)' : 'var(--cyan-500)', color: '#000', opacity: !pw ? 0.5 : 1 }}>
           {loading ? <Loader2 size={15} className='animate-spin' /> : null}
           {loading ? 'Entrando...' : 'Entrar'}
         </button>
       </form>
     
   );
 }
|
|**04**| **Tela 2 — Visão Geral (Home)**
 /  •  KPIs, alertas e agenda do dia
|
| :-: | :- |

## **4.1 — Wireframe**
**▸ WIREFRAME — Visão Geral / Home**
```text
┌─ TOPBAR ──────────────────────────────────────────────────────────────┐
│  Visão Geral   Quinta, 19 de Fevereiro                  [avatar] [⚙]  │
├───────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │ CLIENTES │ │ VENCEM   │ │ EM ATRASO│ │ RECEITA  │  ← StatCards   │
│  │  Ativos  │ │ 7 dias   │ │          │ │  /mês    │                │
│  │    28    │ │    5     │ │    3     │ │ R$840    │                │
│  │  +2 mês  │ │ esta sem.│ │ ▲ 1 novo │ │ ▲ 5%    │                │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘                │
│                                                                        │
│  ┌─ AGENDA DO DIA ──────────────────┐  ┌─ ALERTAS ────────────────┐  │
│  │ Cobranças enviadas hoje: 3        │  │ 🔴 Carlos — 5 dias atraso│  │
│  │                                  │  │ 🟡 Ana — vence amanhã    │  │
│  │  ● João Silva     [✓ enviado]    │  │ 🟡 Pedro — vence amanhã  │  │
│  │  ● Maria Santos   [✓ enviado]    │  │                          │  │
│  │  ● Roberto Lima   [✓ enviado]    │  │ [ Ver todos os alertas ] │  │
│  │                                  │  └──────────────────────────┘  │
│  │  Próximas cobranças: amanhã (2)  │                                 │
│  └──────────────────────────────────┘                                 │
│                                                                        │
│  ┌─ ATIVIDADE RECENTE (últimos pagamentos) ───────────────────────┐   │
│  │ [avatar] João Silva    Mensal    R$30   pago há 2h  ✓ Ativo   │   │
│  │ [avatar] Ana Costa     Mensal    R$30   pago há 5h  ✓ Ativo   │   │
│  └──────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

## **4.2 — KPIs (StatCards)**
|**Card**|**Ícone**|**Valor (query)**|**Cor**|
| :- | :- | :- | :- |
|Clientes Ativos|Users|COUNT(status = ACTIVE)|--cyan-500|
|Vencem em 7 dias|CalendarClock|COUNT(dueDate BETWEEN hoje e +7d)|--yellow|
|Em Atraso|AlertTriangle|COUNT(status = OVERDUE)|--red|
|Receita do Mês|TrendingUp|SUM(amount) WHERE paidAt >= início do mês|--green|

## **4.3 — Prompt para a IA Gerar a Tela**
**🤖  PROMPT — PROMPT — src/pages/HomePage.jsx**
```text

```
| Crie src/pages/HomePage.jsx para o Itaflix.
 Use React Query para buscar dados de GET /api/dashboard/summary que retorna:
 { activeClients, dueSoon, overdue, monthRevenue, todayBillings, recentPayments, alerts }
 Layout:
   1. Topbar interna: título 'Visão Geral' à esquerda + data de hoje formatada à direita
   2. Grid 2x2 de StatCards (componente StatCard de src/components/ui/StatCard.jsx):
      - Clientes Ativos (ícone Users, cor cyan)
      - Vencem em 7 dias (ícone CalendarClock, cor yellow)
      - Em Atraso (ícone AlertTriangle, cor red)
      - Receita do Mês com prefixo R$ (ícone TrendingUp, cor green)
   3. Linha de dois cards (lado a lado):
      Card esquerdo 'Agenda do Dia': lista de cobranças enviadas hoje com check verde.
      Se nenhuma: EmptyState 'Nenhuma cobrança enviada hoje'.
      Card direito 'Alertas': lista de clientes em atraso + clientes que vencem amanhã.
      Cada alerta tem cor diferente (vermelho para atraso, amarelo para amanhã).
      Botão 'Ver cliente' que navega para /clientes/:id.
   4. Card 'Atividade Recente': tabela compacta dos últimos 5 pagamentos.
      Colunas: Avatar+Nome, Plano, Valor, 'há X horas', Badge de status.
 Loading: use Skeleton nos 4 StatCards enquanto carrega.
 Use os CSS variables definidos em tokens.css para todas as cores.
|
|**05**| **Tela 3 — Clientes**
 /clientes  •  Listagem, busca, filtros e ações
|
| :-: | :- |

## **5.1 — Wireframe**
**▸ WIREFRAME — Tela de Clientes**
```text
┌─ TOPBAR ─────────────────────────────────────────────── [+ Novo Cliente] ─┐
│  Clientes   28 registros                                                   │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [🔍 Buscar por nome ou telefone...] [Status ▾] [Plano ▾]  [Exportar CSV] │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │  CLIENTE          TELEFONE      PLANO       VENCIMENTO  STATUS AÇÃO │  │
│  │  ─────────────────────────────────────────────────────────────────  │  │
│  │  [◉] João Silva   21 9 9999-0000 🔵 Mensal  15/03 ✓    ● Ativo  ⋮  │  │
│  │  [◉] Ana Costa    21 9 8888-0000 🔵 Mensal  16/03 ✓    ● Ativo  ⋮  │  │
│  │  [◉] Carlos Lima  21 9 7777-0000 🟣 Trim.   10/02 !!   ● Atraso ⋮  │  │
│  │  [◉] Maria Souza  21 9 6666-0000 🔵 Mensal  20/03 →    ● Ativo  ⋮  │  │
│  │  ...                                                                │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ← 1  2  3 →    Mostrando 1-15 de 28                                      │
└─────────────────────────────────────────────────────────────────────────────┘

Menu ⋮ (ao clicar):
┌─────────────────────┐
│ 👁 Ver detalhes     │
│ ✏ Editar           │
│ 📲 Enviar cobrança │
│ ─────────────────  │
│ 🚫 Desativar       │
└─────────────────────┘
```

## **5.2 — Lógica de Cores na Coluna Vencimento**
|**Condição**|**Cor / Ícone**|**Exemplo visual**|
| :- | :- | :- |
|Vence em > 7 dias|Texto muted normal|15/03/2025|
|Vence em 4–7 dias|Amarelo claro|15/03 →|
|Vence em 1–3 dias|Amarelo intenso + pulsando|15/03 ⚡|
|Vence amanhã|Laranja + badge AMANHÃ|Amanhã !!|
|Vencido (atrasado)|Vermelho + dias em atraso|10/02 (5d atraso)|

## **5.3 — Hook de Filtros (Reutilizável)**
**📄 src/hooks/useClientFilters.js**
```javascript

```
| import { useState, useMemo } from 'react';
 export function useClientFilters(clients = []) {
   const [search, setSearch] = useState('');
   const [status, setStatus] = useState('all');
   const [plan,   setPlan]   = useState('all');
   const [page,   setPage]   = useState(1);
   const PER\_PAGE = 15;
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
   const paginated  = filtered.slice((page - 1) \* PER\_PAGE, page \* PER\_PAGE);
   const totalPages = Math.ceil(filtered.length / PER\_PAGE);
   return { search, setSearch, status, setStatus, plan, setPlan,
            page, setPage, paginated, totalPages, total: filtered.length };
 }
|

## **5.4 — Prompt para a IA Gerar a Tela**
**🤖  PROMPT — PROMPT — src/pages/ClientsPage.jsx**
```text

```
| Crie src/pages/ClientsPage.jsx para o Itaflix.
 Dados: GET /api/clients (retorna todos os clientes com campo daysUntilDue calculado).
 Use React Query com staleTime: 60000.
 Filtros (use o hook useClientFilters de src/hooks/useClientFilters.js):
   - Input de busca com ícone Search do lucide-react
   - Select status: Todos / Ativo / Em atraso / Inativo
   - Select plano: Todos / Mensal / Trimestral / Semestral / Anual
 Tabela com colunas:
   Nome (Avatar + nome + email em cinza abaixo) | Telefone formatado | Plano (Badge) |
   Vencimento (cor dinâmica conforme daysUntilDue) | Status (Badge) | Ações (menu ⋮)
 Menu de ações (Dropdown ao clicar em ⋮):
   - Ver detalhes → navega /clientes/:id
   - Editar → navega /clientes/:id/editar
   - Enviar cobrança → chama POST /api/clients/:id/send-billing, mostra toast
   - Desativar → confirm dialog, depois PUT /api/clients/:id com status INACTIVE
 Paginação: 15 por página, mostrar '1–15 de 28'.
 Loading: Skeleton de 8 linhas na tabela.
 Empty state: se nenhum resultado nos filtros, mostrar EmptyState com mensagem.
 Botão 'Novo Cliente' no topbar → navega para /clientes/novo.
 Botão 'Exportar CSV' que gera e baixa um .csv com os dados filtrados.
|
|**06**| **Tela 4 — Novo / Editar Cliente**
 /clientes/novo  e  /clientes/:id/editar
|
| :-: | :- |

## **6.1 — Wireframe**
**▸ WIREFRAME — Formulário de Cliente (Novo / Editar)**
```text
┌─ TOPBAR ───────────────────────────────────────────────────────────────┐
│  ← Clientes  /  Novo cliente                                           │
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─ DADOS PESSOAIS ─────────────────────┐  ┌─ PLANO E VENCIMENTO ──┐  │
│  │                                      │  │                        │  │
│  │  [Nome completo             ]        │  │  Plano:                │  │
│  │                                      │  │  ┌─────────────────┐   │  │
│  │  [Telefone (99) 99999-9999  ]        │  │  │ ◉ Mensal  R$30 │   │  │
│  │                                      │  │  │ ○ Trimestral   │   │  │
│  │  [E-mail (opcional)         ]        │  │  │ ○ Semestral    │   │  │
│  │                                      │  │  │ ○ Anual        │   │  │
│  └──────────────────────────────────────┘  │  └─────────────────┘   │  │
│                                             │                        │  │
│                                             │  Data de vencimento:   │  │
│                                             │  [📅 DD/MM/AAAA    ]  │  │
│                                             │                        │  │
│                                             │  ┌─ RESUMO ─────────┐  │  │
│                                             │  │ Mensal • R$ 30   │  │  │
│                                             │  │ Vence em 30 dias │  │  │
│                                             │  └──────────────────┘  │  │
│                                             └────────────────────────┘  │
│                                                                          │
│  [Cancelar]                                         [Salvar cliente →]  │
└──────────────────────────────────────────────────────────────────────────┘
```

## **6.2 — Validações**
|**Campo**|**Regra**|**Mensagem de erro**|
| :- | :- | :- |
|Nome|Obrigatório, mínimo 3 chars|Nome precisa ter ao menos 3 caracteres|
|Telefone|11 dígitos após limpar máscara|Telefone inválido — use (DDD) + 9 dígitos|
|Telefone|Único no sistema|Esse telefone já está cadastrado|
|Plano|Um dos 4 planos válidos|Selecione um plano|
|Vencimento|Obrigatório, não pode ser passado (em novo cadastro)|Data de vencimento inválida|

## **6.3 — Prompt para a IA Gerar a Tela**
**🤖  PROMPT — PROMPT — src/pages/NewClientPage.jsx**
```text

```
| Crie src/pages/NewClientPage.jsx para o Itaflix.
 Se a rota contiver um :id (edição), carregue os dados via GET /api/clients/:id
 e preencha o formulário. O título deve mudar para 'Editar cliente'.
 Layout em duas colunas (no desktop), uma coluna no mobile:
 Coluna esquerda — Dados pessoais:
   - Input Nome completo (obrigatório)
   - Input Telefone com máscara (99) 99999-9999 — instale react-input-mask
   - Input E-mail (opcional, type email)
 Coluna direita — Plano e vencimento:
   - Seleção de plano como radio cards visuais (não select comum).
     Cada opção é um card clicável mostrando nome e preço.
     Ao selecionar, o card fica destacado com borda ciano.
   - Date picker nativo (type='date') para data de vencimento.
   - Card de resumo (ciano claro) mostrando: plano selecionado, valor, e 'vence em X dias'.
     Atualiza em tempo real conforme o usuário muda os campos.
 Validação em tempo real: erros aparecem abaixo do campo ao sair do foco (onBlur).
 Botão Salvar desabilitado se houver erros ou campos obrigatórios vazios.
 Ao salvar:
   Novo: POST /api/clients → toast 'Cliente cadastrado!' → navega /clientes
   Editar: PUT /api/clients/:id → toast 'Cliente atualizado!' → navega /clientes/:id
 Botão Cancelar volta para a tela anterior (navigate(-1)).
|
|**07**| **Tela 5 — Detalhe do Cliente**
 /clientes/:id  •  Ficha completa
|
| :-: | :- |

## **7.1 — Wireframe**
**▸ WIREFRAME — Detalhe do Cliente**
```text
┌─ TOPBAR ──────────────────────────────────────────────────────────────┐
│  ← Clientes  /  João Silva                [Editar]  [Enviar cobrança]│
├───────────────────────────────────────────────────────────────────────┤
│                                                                        │
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
│  │  ABA: HISTÓRICO                                                │  │
│  │  DATA         VALOR   NOVA VALIDADE    STATUS                  │  │
│  │  13/02/2025  R$30    15/03/2025       ✓ Confirmado            │  │
│  │  15/01/2025  R$30    14/02/2025       ✓ Confirmado            │  │
│  │  ...                                                           │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                        │
│  ABA: NOTAS                                                           │
│  ┌──────────────────────────────────────────────────────────────────┐│
│  │  [Escreva uma nota sobre este cliente...             ] [Salvar] ││
│  │  ─────────────────────────────────────────────────────────────  ││
│  │  10/02 — 'Cliente pediu para mudar para plano trimestral'       ││
│  │  05/01 — 'Pagou adiantado, bem engajado'                        ││
│  └──────────────────────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────────────────────────┘
```

## **7.2 — Prompt para a IA Gerar a Tela**
**🤖  PROMPT — PROMPT — src/pages/ClientDetailPage.jsx**
```text

```
| Crie src/pages/ClientDetailPage.jsx para o Itaflix.
 Dados: GET /api/clients/:id (inclui payments[] e notes[] ordenados por data desc).
 Topbar interna:
   - Breadcrumb: ← Clientes / {nome do cliente}
   - Botões à direita: 'Editar' (→ /clientes/:id/editar) e 'Enviar cobrança'
   - 'Enviar cobrança': chama POST /clients/:id/send-billing, mostra toast de confirmação
 Card do cliente (topo da página):
   - Avatar grande com inicial, cor gerada pelo nome (hash simples)
   - Nome, Badge de status, Badge de plano
   - Linha com: telefone (ícone Phone), email (ícone Mail)
   - Linha com: data de vencimento + 'em X dias' (cor dinâmica como na tabela)
   - Linha com: 'Cliente desde' (data de criação formatada)
 Sistema de abas (Histórico de Pagamentos | Notas):
   Aba Histórico:
     Tabela: Data do pagamento | Valor (R$) | Nova validade gerada | ícone ✓
     Se vazio: EmptyState 'Nenhum pagamento registrado ainda'
   Aba Notas:
     Textarea para nova nota + botão Salvar
     POST /api/notes com { clientId, content }
     Lista de notas abaixo, cada uma com data relativa (ex: 'há 3 dias')
     Botão de deletar nota (ícone lixeira, confirmação simples)
 Use date-fns para formatação e cálculos de data.
 Invalide a query do cliente após salvar nota ou enviar cobrança.
|
|**08**| **Tela 6 — Pagamentos**
 /pagamentos  •  Histórico financeiro completo
|
| :-: | :- |

## **8.1 — Wireframe**
**▸ WIREFRAME — Histórico de Pagamentos**
```text
┌─ TOPBAR ─────────────────────────────────────────────────────────────────┐
│  Pagamentos   R$ 840 recebido em Fevereiro                               │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│  │ Este mês     │  │ Mês anterior │  │ Total geral  │  ← mini KPIs     │
│  │ R$ 840       │  │ R$ 780       │  │ R$ 8.400     │                  │
│  └──────────────┘  └──────────────┘  └──────────────┘                  │
│                                                                           │
│  [🔍 Buscar cliente...]  [Período: Este mês ▾]  [Plano ▾]               │
│                                                                           │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  DATA         CLIENTE        PLANO    VALOR   NOVA VALIDADE        │  │
│  │  ──────────────────────────────────────────────────────────────    │  │
│  │  13/02 14:32  João Silva     Mensal   R$30    15/03/2025          │  │
│  │  13/02 10:15  Ana Costa      Mensal   R$30    16/03/2025          │  │
│  │  12/02 18:44  Carlos Lima    Trimest. R$80    11/05/2025          │  │
│  │  ...                                                               │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│  Total filtrado: R$ 840 (28 pagamentos)                                   │
└───────────────────────────────────────────────────────────────────────────┘
```

## **8.2 — Prompt para a IA Gerar a Tela**
**🤖  PROMPT — PROMPT — src/pages/PaymentsPage.jsx**
```text

```
| Crie src/pages/PaymentsPage.jsx para o Itaflix.
 Dados: GET /api/payments (retorna pagamentos com dados do cliente incluídos).
 Topbar: título 'Pagamentos' + valor total do mês em destaque à direita.
 3 mini StatCards no topo:
   - Este mês: soma dos pagamentos do mês atual
   - Mês anterior: soma do mês passado
   - Total geral: soma de tudo
 Filtros:
   - Input de busca por nome do cliente
   - Select período: Esta semana / Este mês / Mês anterior / 3 meses / Tudo
   - Select plano: Todos / Mensal / Trimestral / Semestral / Anual
 Tabela com colunas:
   Data e hora (formato DD/MM às HH:mm) | Cliente (Avatar+Nome) | Plano (Badge) |
   Valor (verde, fonte mono) | Nova validade gerada
 Rodapé da tabela: 'Total filtrado: R$ X (N pagamentos)'
 Ao clicar em uma linha da tabela, navegar para /clientes/:id do cliente.
 Use date-fns para todos os cálculos de período e formatações.
 Loading: Skeleton de 8 linhas.
|
|**09**| **Roteamento, Guards e App.jsx**
 Estrutura completa de navegação
|
| :-: | :- |

## **9.1 — Código Completo do App.jsx**
**📄 src/App.jsx**
```jsx

```
| import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
 import { QueryClientProvider } from '@tanstack/react-query';
 import { queryClient } from './lib/queryClient';
 import { Toaster } from './components/ui/Toast';
 import { Layout }          from './components/Layout';
 import LoginPage           from './pages/LoginPage';
 import HomePage            from './pages/HomePage';
 import ClientsPage         from './pages/ClientsPage';
 import NewClientPage       from './pages/NewClientPage';
 import ClientDetailPage    from './pages/ClientDetailPage';
 import PaymentsPage        from './pages/PaymentsPage';
 function PrivateRoute({ children }) {
   return localStorage.getItem('itaflix\_token') ? children : <Navigate to='/login' replace />;
 }
 export default function App() {
   return (
     <QueryClientProvider client={queryClient}>
       <BrowserRouter>
         <Routes>
           <Route path='/login' element={<LoginPage />} />
           <Route path='/' element={ <Layout /></PrivateRoute>}>
             <Route index        element={<HomePage />} />
             <Route path='clientes'            element={<ClientsPage />} />
             <Route path='clientes/novo'       element={<NewClientPage />} />
             <Route path='clientes/:id'        element={<ClientDetailPage />} />
             <Route path='clientes/:id/editar' element={<NewClientPage />} />
             <Route path='pagamentos'          element={ } />
           </Route>
           <Route path='\*' element={<Navigate to='/' replace />} />
         </Routes>
         <Toaster />
       </BrowserRouter>
     </QueryClientProvider>
   );
 }
|

## **9.2 — Cliente HTTP (api.js)**
**📄 src/lib/api.js**
```javascript

```
| import axios from 'axios';
 export const api = axios.create({
   baseURL: import.meta.env.VITE\_API\_URL,
 });
 // Injeta o token JWT em todas as requisições
 api.interceptors.request.use(config => {
   const token = localStorage.getItem('itaflix\_token');
   if (token) config.headers.Authorization = `Bearer ${token}`;
   return config;
 });
 // Se receber 401, redireciona para o login
 api.interceptors.response.use(
   res => res,
   err => {
     if (err.response?.status === 401) {
       localStorage.removeItem('itaflix\_token');
       window.location.href = '/login';
     }
     return Promise.reject(err);
   }
 );
|
|**10**| **Checklist de Implementação da Dashboard**
 Ordem recomendada para construir tudo
|
| :-: | :- |

## **Fase A — Fundação (faça antes de qualquer tela)**
| |**Tarefa**|
| :- | :- |
|☐|Criar projeto com Vite: npm create vite@latest itaflix-dashboard -- --template react|
|☐|Instalar dependências: axios, react-router-dom, @tanstack/react-query, lucide-react, date-fns|
|☐|Instalar e configurar TailwindCSS com postcss e autoprefixer|
|☐|Criar src/styles/tokens.css com todas as CSS variables da paleta|
|☐|Importar fontes DM Mono + DM Sans no index.html|
|☐|Criar src/lib/api.js com instância axios + interceptors de token e 401|
|☐|Criar src/lib/queryClient.js com configuração do React Query|
|☐|Criar src/App.jsx com roteamento completo e PrivateRoute|
|☐|Criar todos os componentes de src/components/ui/ (Badge, Button, Card, StatCard...)|
|☐|Criar src/components/Layout.jsx com sidebar + outlet|
|☐|Criar arquivo .env.local com VITE\_API\_URL|

## **Fase B — Telas (nesta ordem)**
||**Tela**|**Por que essa ordem**|
| :- | :- | :- |
|☐|**Login**|É independente — não precisa de dados reais para testar|
|☐|**Visão Geral**|Valida que o backend e as queries funcionam antes de tudo|
|☐|**Novo Cliente**|Precisa existir para popular dados para testar o restante|
|☐|**Clientes**|Depende de clientes cadastrados para ver filtros e paginação|
|☐|**Detalhe Cliente**|Depende de clientes com histórico para testar as abas|
|☐|**Pagamentos**|Depende de pagamentos reais para ter dados significativos|
|☐|**Calendário**|Por último: integra tudo — cobranças, tarefas e recorrências|
|**11**| **Calendário — Modelo de Dados e Tipos de Evento**
 Schema, categorias, recorrência e integração com WhatsApp
|
| :-: | :- |

## **11.1 — Os Três Tipos de Evento**
O calendário unifica três origens diferentes de eventos em uma única visualização:
||**Tipo**|**Origem**|**Exemplos**|
| :- | :- | :- | :- |
|🔵|**Cobrança**|Gerado automaticamente pelo sistema|Vencimento de assinatura de cliente (mensal, trimestral, etc.)|
|🟣|**Tarefa única**|Criada por você manualmente ou via WhatsApp|Reunião dia 12/06 às 19h, Pagar imposto em 12/04 às 13h|
|🟡|**Tarefa recorrente**|Criada por você com regra de repetição|Tomar remédio toda sexta às 12h, Pagar internet todo dia 15|

## **11.2 — Schema Prisma (novas tabelas)**
Adicione ao prisma/schema.prisma existente as tabelas de eventos e recorrências:
**📄 prisma/schema.prisma — adicionar ao schema existente**
```prisma

```
| // Enum de frequência de recorrência
 enum RecurrenceFreq {
   DAILY       // todo dia
   WEEKLY      // toda semana (usa weekDay)
   MONTHLY\_DAY // todo dia X do mês (usa monthDay)
   YEARLY      // todo ano nessa data
   ONCE        // evento único (sem recorrência)
 }
 // Categoria visual do evento
 enum EventCategory {
   BILLING   // cobrança de cliente — gerado pelo sistema
   TASK      // tarefa pessoal única
   RECURRING // tarefa recorrente
   REMINDER  // lembrete avulso (agendado via WhatsApp/IA)
 }
 model CalendarEvent {
   id          Int             @id @default(autoincrement())
   title       String          // ex: 'Cobrança João Silva' ou 'Pagar internet'
   description String?         // detalhes opcionais
   category    EventCategory
   startAt     DateTime        // data e hora do evento / primeiro disparo
   amount      Decimal?        // valor (opcional, útil para cobranças e despesas)
   notifyWhatsApp Boolean      @default(true)
   notifyAt    DateTime?       // quando enviar a notificação (null = no startAt)
   notified    Boolean         @default(false)
   done        Boolean         @default(false)
   // Relação opcional com cliente (para cobranças automáticas)
   clientId    Int?
   client      Client?         @relation(fields: [clientId], references: [id])
   // Relação com regra de recorrência (null = evento único)
   recurrenceId Int?
   recurrence   Recurrence?    @relation(fields: [recurrenceId], references: [id])
   createdAt   DateTime        @default(now())
 }
 model Recurrence {
   id        Int            @id @default(autoincrement())
   freq      RecurrenceFreq
   weekDay   Int?           // 0=Dom, 1=Seg ... 6=Sáb (para WEEKLY)
   monthDay  Int?           // 1-31 (para MONTHLY\_DAY)
   startDate DateTime       // a partir de quando a recorrência vale
   endDate   DateTime?      // null = sem fim
   events    CalendarEvent[]
   createdAt DateTime       @default(now())
 }
|
**💡  Quando uma recorrência é criada, o scheduler gera os próximos 3 meses de eventos de uma vez. A cada semana ele verifica e cria os eventos do mês seguinte automaticamente.**
```text

```

## **11.3 — Exemplos Mapeados para o Schema**
|**Exemplo do usuário**|**category**|**freq**|**weekDay / monthDay**|**notifyAt**|
| :- | :- | :- | :- | :- |
|Reunião dia 12/06/2026 às 19h|REMINDER|ONCE|—|1h antes|
|Pagar imposto em 12/04/2026 às 13h|TASK|ONCE|—|1d antes|
|Remédio toda sexta-feira às 12h|RECURRING|WEEKLY|weekDay: 5|no horário|
|Pagar internet todo dia 15 — R$100|RECURRING|MONTHLY\_DAY|monthDay: 15|1d antes|
|Cobrança João Silva (mensal)|BILLING|ONCE\*|—|1d antes|
**ℹ️  (\*) Cobranças são criadas como ONCE porque cada renovação gera um novo evento com nova data. O sistema as cria automaticamente ao processar o webhook de pagamento.**
```text

```
|**12**| **Calendário — Wireframes das Duas Views**
 Visão mensal e visão semanal/agenda
|
| :-: | :- |

## **12.1 — View Mensal**
**▸ WIREFRAME — Calendário — View Mensal**
```text
┌─ TOPBAR ──────────────────────────────────────────────────────────────────┐
│  Calendário              [← Fev]  Março 2026  [Abr →]   [Mês] [Semana]  │
│                                                          [+ Nova Tarefa]  │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ─────── LEGENDA ──────────────────────────────────────────────────────   │
│  🔵 Cobrança   🟣 Tarefa única   🟡 Recorrente   🔴 Atrasado            │
│                                                                             │
│  Dom    Seg    Ter    Qua    Qui    Sex    Sáb                            │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                      │
│  │    │ │    │ │  3 │ │  4 │ │  5 │ │  6 │ │  7 │                      │
│  │    │ │    │ │    │ │🔵·2│ │    │ │🟡  │ │    │                      │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘                      │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                      │
│  │  8 │ │  9 │ │ 10 │ │ 11 │ │ 12 │ │ 13 │ │ 14 │                      │
│  │    │ │    │ │    │ │    │ │🔵·3│ │🟡  │ │    │                      │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘                      │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                      │
│  │ 15 │ │ 16 │ │ 17 │ │ 18 │ │ 19 │ │ 20 │ │ 21 │                      │
│  │🟡·1│ │    │ │    │ │    │ │🔵·4│ │🟡  │ │    │                      │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘ └────┘                      │
│                    ...continua...                                          │
│                                                                             │
│  → Ao clicar em qualquer dia: abre painel lateral com eventos do dia      │
│  → Ao clicar em um evento: abre modal de detalhe/edição                  │
│  → Dias com eventos em atraso ficam com fundo vermelho escuro             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## **12.2 — View Semanal / Agenda**
**▸ WIREFRAME — Calendário — View Semanal (grade por hora)**
```text
┌─ TOPBAR ─────────────────────────────── [← Semana]  09–15 Mar  [→]  [Mês][Semana] ─┐
│                                                                [+ Nova Tarefa]        │
├──────────────────────────────────────────────────────────────────────────────────────┤
│       Dom 9  Seg 10  Ter 11  Qua 12  Qui 13  Sex 14  Sáb 15                         │
│  ────────────────────────────────────────────────────────────────────────────────    │
│  08h                                                                                  │
│  09h           ┌──────────────┐                                                       │
│                │ 🔵 Cobrança  │                                                       │
│  10h           │ Ana Costa    │                                                       │
│                │ R$ 30 · Mens.│                                                       │
│  11h           └──────────────┘                                                       │
│  12h                                    ┌────────┐  ┌────────────────┐               │
│                                         │🟣Reunião│  │🟡 Remédio     │               │
│  13h                                    │19h     │  │ toda sexta 12h│               │
│                                         └────────┘  └────────────────┘               │
│  14h                                                                                  │
│  15h  ┌──────────────────────┐                                                        │
│       │ 🟡 Pagar internet    │                                                        │
│  16h  │ R$ 100 · todo dia 15 │                                                        │
│       └──────────────────────┘                                                        │
│  ...                                                                                   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

## **12.3 — Painel Lateral de Eventos do Dia**
**▸ WIREFRAME — Painel lateral — Eventos do dia selecionado**
```text
┌─ Eventos — Sexta, 14 de Março ──────────────────┐
│                                  [+ Adicionar]   │
│  ─── COBRANÇAS ────────────────────────────────  │
│  🔵 09:00  Cobrança — João Silva                │
│            Mensal · R$30 · [✓ Enviada] [Ver →]  │
│                                                   │
│  🔵 09:00  Cobrança — Maria Souza               │
│            Anual · R$280 · [Enviar agora]        │
│                                                   │
│  ─── TAREFAS ──────────────────────────────────  │
│  🟡 12:00  Remédio (recorrente · toda sexta)    │
│            [✓ Marcar feito]                      │
│                                                   │
│  ─── LEMBRETES ────────────────────────────────  │
│  🟣 19:00  Reunião com fornecedor                │
│            [✓ Marcar feito]  [Editar]  [🗑]     │
│                                                   │
│  NOTIFICAÇÕES PENDENTES HOJE: 2                  │
│  [Disparar todas agora]                          │
└──────────────────────────────────────────────────┘
```
|**13**| **Calendário — Backend: Rotas e Scheduler**
 API REST + geração automática de eventos recorrentes
|
| :-: | :- |

## **13.1 — Rotas da API de Eventos**
||**Método + Rota**|**Auth**|**Descrição**|
| :- | :- | :- | :- |
|**GET**|/api/events|JWT|Lista eventos com filtros: ?month=3&year=2026 ou ?week=2026-03-09|
|**GET**|/api/events/:id|JWT|Detalhe de um evento específico|
|**POST**|/api/events|JWT|Criar evento único (TASK ou REMINDER)|
|**POST**|/api/events/recurring|JWT|Criar evento recorrente + gera próximas ocorrências|
|**PUT**|/api/events/:id|JWT|Editar evento (titulo, hora, notifyAt, amount)|
|**PATCH**|/api/events/:id/done|JWT|Marcar evento como concluído|
|**DELETE**|/api/events/:id|JWT|Apaga evento único ou uma ocorrência da recorrência|
|**DELETE**|/api/events/:id/all|JWT|Apaga evento e todas as ocorrências futuras da recorrência|
|**POST**|/api/events/:id/notify|JWT|Disparar notificação WhatsApp manualmente para um evento|

## **13.2 — Serviço de Geração de Recorrências**
**📄 src/services/recurrence.js**
```javascript

```
| import { addDays, addWeeks, addMonths, addYears, setDate, getDay } from 'date-fns';
 /\*\*
  \* Gera as datas de ocorrência de uma recorrência
  \* a partir de hoje até 'horizonDays' dias à frente.
  \*/
 export function generateOccurrences(recurrence, horizonDays = 90) {
   const { freq, weekDay, monthDay, startDate, endDate } = recurrence;
   const occurrences = [];
   const horizon     = addDays(new Date(), horizonDays);
   let   cursor      = new Date(Math.max(new Date(startDate), new Date()));
   while (cursor <= horizon && (!endDate || cursor <= new Date(endDate))) {
     occurrences.push(new Date(cursor));
     switch (freq) {
       case 'DAILY':       cursor = addDays(cursor, 1);   break;
       case 'WEEKLY':      cursor = addWeeks(cursor, 1);  break;
       case 'MONTHLY\_DAY': cursor = addMonths(cursor, 1); break;
       case 'YEARLY':      cursor = addYears(cursor, 1);  break;
       default: return occurrences; // ONCE — apenas uma ocorrência
     }
   }
   return occurrences;
 }
 /\*\*
  \* Cria os CalendarEvents no banco para cada ocorrência
  \* Evita duplicatas verificando startAt + recurrenceId já existentes
  \*/
 export async function materializeRecurrence(prisma, recurrence, eventTemplate) {
   const dates = generateOccurrences(recurrence);
   const created = [];
   for (const date of dates) {
     const exists = await prisma.calendarEvent.findFirst({
       where: { recurrenceId: recurrence.id, startAt: date }
     });
     if (exists) continue;
     const notifyAt = eventTemplate.notifyAt
       ? addDays(date, -1)   // padrão: notificar 1 dia antes
       : date;               // ou no horário do evento
     created.push(await prisma.calendarEvent.create({
       data: { ...eventTemplate, startAt: date, notifyAt, recurrenceId: recurrence.id }
     }));
   }
   return created;
 }
|

## **13.3 — Scheduler: Notificações de Eventos**
Adicione estes dois jobs ao src/services/scheduler.js existente:
**📄 src/services/scheduler.js — adicionar ao arquivo existente**
```javascript

```
| // JOB 3: Notificações de eventos do calendário (roda a cada 5 minutos)
 cron.schedule('\*/5 \* \* \* \*', async () => {
   const now  = new Date();
   const soon = addMinutes(now, 5);
   const pending = await prisma.calendarEvent.findMany({
     where: {
       notifyWhatsApp: true,
       notified:       false,
       done:           false,
       notifyAt:       { gte: now, lte: soon },
     },
     include: { client: true },
   });
   for (const event of pending) {
     try {
       await sendEventNotification(event);
       await prisma.calendarEvent.update({
         where: { id: event.id },
         data:  { notified: true },
       });
       console.log(`Notificação enviada: [${event.category}] ${event.title}`);
     } catch (err) {
       console.error(`Erro ao notificar evento ${event.id}:`, err.message);
     }
   }
 });
 // JOB 4: Expandir recorrências (roda toda segunda às 07:00)
 // Garante que o horizonte de 90 dias sempre esteja populado
 cron.schedule('0 7 \* \* 1', async () => {
   const recurrences = await prisma.recurrence.findMany({
     where: { OR: [{ endDate: null }, { endDate: { gte: new Date() } }] }
   });
   for (const rec of recurrences) {
     const tmpl = await prisma.calendarEvent.findFirst({ where: { recurrenceId: rec.id } });
     if (tmpl) await materializeRecurrence(prisma, rec, tmpl);
   }
   console.log(`Recorrências expandidas: ${recurrences.length} regras processadas`);
 });
|

## **13.4 — Serviço: Mensagens de Notificação por Categoria**
**📄 src/services/whatsapp.js — adicionar função sendEventNotification**
```javascript

```
| export async function sendEventNotification(event) {
   const date = format(new Date(event.startAt), "dd/MM/yyyy 'às' HH'h'mm", { locale: ptBR });
   let text = '';
   switch (event.category) {
     case 'BILLING':
       // Enviada para o CLIENTE — event.client deve existir
       text = `💳 \*Itaflix — Cobrança\*\n\n
              Olá, \*${event.client.name}\*!\n
              Sua assinatura vence amanhã, \*${date}\*.\n\n
              💰 Valor: R$ ${event.amount}\n
              🔗 ${event.pixLink || 'Em breve'}`;
       return sendMessage(event.client.phone, text);
     case 'TASK':
     case 'REMINDER':
       // Enviada para VOCÊ
       text = `🔔 \*Lembrete Itaflix\*\n\n
              📌 ${event.title}\n
              🕐 ${date}
              `(event.description ? `\n📝 ${event.description}` : '')
              (event.amount ? `\n💰 R$ ${event.amount}` : '');
       return sendMessage(process.env.ADMIN\_PHONE, text);
     case 'RECURRING':
       text = `🔁 \*Tarefa recorrente\*\n\n
              📌 ${event.title}\n
              🕐 ${date}
              `(event.amount ? `\n💰 R$ ${event.amount}` : '');
       return sendMessage(process.env.ADMIN\_PHONE, text);
   }
 }
|
|**14**| **Calendário — Frontend: Tela Completa**
 React com visão mensal, semanal e modal de criação
|
| :-: | :- |

## **14.1 — Dependências necessárias**
**📄 bash**
```bash

```
| # Calendário visual (leve e altamente customizável)
 npm install @fullcalendar/react @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction
 # Alternativa mais simples (sem FullCalendar):
 # npm install react-big-calendar date-fns
 # Para os date pickers do modal de criação:
 npm install react-datepicker
|
**💡  Recomendação: use @fullcalendar porque suporta nativo views de mês e semana, drag-and-drop de eventos, e é fácil de estilizar com CSS variables. A licença open source é suficiente para este projeto.**
```text

```

## **14.2 — Prompt para a IA Gerar a Tela do Calendário**
**🤖  PROMPT — PROMPT — src/pages/CalendarPage.jsx (parte 1 — estrutura e dados)**
```text

```
| Crie src/pages/CalendarPage.jsx para o Itaflix.
 Dados: GET /api/events?month={m}&year={y} retorna array de CalendarEvent.
 Use React Query: queryKey ['events', year, month], refetchOnWindowFocus: false.
 Mapeie os eventos para o formato do FullCalendar:
   BILLING:   backgroundColor: '#06B6D4', borderColor: '#0891B2'
   TASK:      backgroundColor: '#8B5CF6', borderColor: '#7C3AED'
   RECURRING: backgroundColor: '#F59E0B', borderColor: '#D97706'
   REMINDER:  backgroundColor: '#EC4899', borderColor: '#DB2777'
   done: true → adicionar className 'fc-event-done' (opacidade 50%)
 Configurações do FullCalendar:
   - plugins: dayGridPlugin, timeGridPlugin, interactionPlugin
   - headerToolbar: false (vamos fazer nosso próprio header)
   - initialView: 'dayGridMonth'
   - locale: 'pt-br'
   - height: 'auto'
   - eventClick: abre modal de detalhe do evento
   - dateClick: abre painel lateral com eventos do dia clicado
   - eventContent: renderiza ícone + título customizado
 Header customizado acima do calendário:
   - Botões ← e → para navegar meses/semanas (usa calendarRef.current.getApi())
   - Título do período atual no centro
   - Toggle entre 'Mês' e 'Semana'
   - Botão '+ Nova Tarefa' que abre modal de criação
 Legenda de cores abaixo do header (pills com as 4 categorias).
|
**🤖  PROMPT — PROMPT — src/pages/CalendarPage.jsx (parte 2 — modais e painel)**
```text

```
| Adicione ao CalendarPage.jsx do Itaflix os seguintes componentes internos:
 1\. Modal de CRIAÇÃO/EDIÇÃO de evento (abre ao clicar '+ Nova Tarefa' ou editar):
    Campos:
    - Título (obrigatório)
    - Tipo: radio buttons — Tarefa única / Tarefa recorrente / Lembrete
    - Data e hora (datetime-local input)
    - Valor em R$ (opcional, placeholder 'ex: 100.00')
    - Descrição (textarea, opcional)
    - Toggle 'Notificar no WhatsApp' (default: ligado)
    - Antecedência da notificação: select — No horário / 30min antes / 1h antes / 1 dia antes
    SE tipo = Tarefa recorrente, mostrar painel extra:
    - Frequência: Diária / Semanal / Mensal (dia do mês) / Anual
    - SE Semanal: checkboxes de dias da semana (Dom/Seg/Ter/Qua/Qui/Sex/Sáb)
    - SE Mensal: input 'Todo dia \_\_\_\_ do mês'
    - Data de término (opcional, placeholder 'Sem data de fim')
    Botões: Cancelar e Salvar
    POST /api/events (único) ou POST /api/events/recurring conforme o tipo.
 2\. Painel lateral (drawer à direita) de eventos do dia clicado:
    - Título: 'Eventos — {data formatada}'
    - Agrupa por categoria: COBRANÇAS / TAREFAS / LEMBRETES
    - Cada evento mostra: hora, título, valor (se houver), botões de ação
    - Botão 'Marcar feito' (PATCH /api/events/:id/done)
    - Botão 'Disparar notificação' (POST /api/events/:id/notify)
    - Para BILLING: mostra nome do cliente e link 'Ver cliente →'
    - Fechar ao clicar fora ou no X
 3\. Modal de DETALHE ao clicar em um evento no calendário:
    - Mostra todos os dados do evento
    - Botão Editar (abre modal de edição)
    - Botão Excluir (para recorrentes: pergunta 'Só este' ou 'Este e todos os futuros')
    - Botão Disparar notificação agora
|

## **14.3 — CSS Global para o FullCalendar**
**📄 src/styles/calendar.css (importar no CalendarPage.jsx)**
```css

```
| /\* Fundo geral do calendário \*/
 .fc { --fc-border-color: var(--border); }
 .fc-theme-standard td, .fc-theme-standard th { border-color: var(--border); }
 .fc-scrollgrid { background: var(--bg-surface); }
 /\* Cabeçalho dos dias da semana \*/
 .fc-col-header-cell { background: var(--bg-panel); }
 .fc-col-header-cell-cushion {
   color: var(--text-secondary);
   font-size: 0.75rem;
   font-weight: 600;
   letter-spacing: 0.05em;
   text-transform: uppercase;
   text-decoration: none;
 }
 /\* Células dos dias \*/
 .fc-daygrid-day { background: var(--bg-surface); }
 .fc-daygrid-day:hover { background: var(--bg-hover); }
 .fc-daygrid-day-number {
   color: var(--text-secondary);
   font-family: var(--font-mono);
   font-size: 0.8rem;
   text-decoration: none;
 }
 /\* Dia atual destacado \*/
 .fc-day-today { background: var(--cyan-950) !important; }
 .fc-day-today .fc-daygrid-day-number { color: var(--cyan-400); font-weight: 700; }
 /\* Eventos \*/
 .fc-event {
   border-radius: 4px;
   font-size: 0.72rem;
   font-weight: 500;
   padding: 1px 4px;
   cursor: pointer;
 }
 .fc-event-done { opacity: 0.4; text-decoration: line-through; }
 /\* Grade de horas (view semanal) \*/
 .fc-timegrid-slot { background: var(--bg-surface); }
 .fc-timegrid-slot-label { color: var(--text-muted); font-family: var(--font-mono); font-size: 0.72rem; }
|

## **14.4 — Adicionar Calendário à Sidebar e ao Roteamento**
**📄 src/components/Layout.jsx — atualizar array NAV**
```jsx

```
| import { LayoutDashboard, Users, CreditCard, Calendar, LogOut, Wifi } from 'lucide-react';
 const NAV = [
   { to: '/',           icon: LayoutDashboard, label: 'Visão Geral'  },
   { to: '/clientes',   icon: Users,           label: 'Clientes'     },
   { to: '/pagamentos', icon: CreditCard,       label: 'Pagamentos'   },
   { to: '/calendario', icon: Calendar,         label: 'Calendário'   },  // ← NOVO
 ];
|
**📄 src/App.jsx — adicionar rota**
```jsx

```
| import CalendarPage from './pages/CalendarPage';
 // Dentro do <Route path='/' element={...}>:
 <Route path='calendario' element={<CalendarPage />} />
|
|**15**| **Integração WhatsApp → Calendário**
 Criar tarefas e lembretes diretamente pelo WhatsApp (Fase IA)
|
| :-: | :- |
Esta seção documenta como a Fase IA (descrita no Vibe Coding guide) se integra especificamente com o módulo de calendário. Com o backend de calendário pronto, o assistente de IA pode criar qualquer tipo de evento diretamente de uma mensagem de texto ou voz.

## **15.1 — Exemplos de Comandos e Resultado**
|**Você escreve/fala**|**Intent (IA detecta)**|**O que o sistema cria**|
| :- | :- | :- |
|Marcar reunião dia 12/06 às 19h|CREATE\_REMINDER|CalendarEvent REMINDER · 12/06 19h · notifyAt: 1h antes|
|Pagar imposto no dia 12/04 às 13h|CREATE\_TASK|CalendarEvent TASK · 12/04 13h · notifyAt: 1d antes|
|Preciso tomar remédio toda sexta às 12h|CREATE\_RECURRING|Recurrence WEEKLY weekDay:5 + eventos gerados p/ 90 dias|
|Pagar internet todo dia 15, R$ 100|CREATE\_RECURRING|Recurrence MONTHLY\_DAY monthDay:15, amount:100|
|Me lembra amanhã às 9h de ligar pro fornecedor|CREATE\_REMINDER|CalendarEvent REMINDER · amanhã 09h · notifyAt: no horário|
|Cancelar a reunião do dia 12|DELETE\_EVENT|Busca e apaga o evento mais próximo com 'reunião' no título|
|Quais eventos tenho essa semana?|LIST\_EVENTS|Responde com lista formatada dos eventos da semana atual|

## **15.2 — Prompt para o aiAssistant.js processar eventos de calendário**
**🤖  PROMPT — PROMPT — Atualizar src/services/aiAssistant.js**
```text

```
| Atualize o system prompt da função processCommand em src/services/aiAssistant.js
 para incluir as intenções de calendário.
 Adicione ao JSON de resposta esperado os campos para eventos:
 Intenções existentes: ADD\_NOTE, LIST\_CLIENTS, REGISTER\_PAYMENT, CREATE\_REMINDER,
                       LIST\_OVERDUE, UNKNOWN
 Novas intenções de calendário:
   CREATE\_TASK      — tarefa única com data específica
   CREATE\_RECURRING — tarefa que se repete (diária/semanal/mensal)
   DELETE\_EVENT     — cancelar um evento pelo título e data
   LIST\_EVENTS      — listar eventos de um período
 Para CREATE\_TASK e CREATE\_REMINDER, o JSON deve retornar:
   { intent, title, datetime (ISO 8601), amount?, description?, notifyBefore }
   notifyBefore: 'now' | '30min' | '1h' | '1day'
 Para CREATE\_RECURRING, o JSON deve retornar:
   { intent, title, freq ('DAILY'|'WEEKLY'|'MONTHLY\_DAY'),
     weekDay?, monthDay?, startDate, amount?, description?, notifyBefore }
 Para datas relativas em português, parse usando date-fns/locale/pt-BR:
   'amanhã' → tomorrow, 'semana que vem' → next Monday,
   'toda sexta' → WEEKLY weekDay:5, 'todo dia 15' → MONTHLY\_DAY monthDay:15
 Após identificar a intenção, chame a rota interna POST /api/events ou
 POST /api/events/recurring com os dados extraídos.
 Responda ao usuário confirmando o que foi criado.
|

# **Checklist do Módulo Calendário**
**💡  Implemente o calendário APÓS o MVP básico estar funcionando. É uma adição que não quebra nada — apenas adiciona novas tabelas e rotas.**
```text
|**Tarefa**|**Depende de**
:- | :- | :-
☐|Adicionar enums e models CalendarEvent + Recurrence ao schema Prisma|Banco existente
☐|Rodar npx prisma db push para criar as tabelas|Schema atualizado
☐|Criar src/services/recurrence.js com generateOccurrences|date-fns instalado
☐|Criar src/routes/events.js com todas as rotas CRUD|Prisma + auth middleware
☐|Adicionar sendEventNotification ao whatsapp.js|Evolution API ativa
☐|Adicionar Job 3 (notificações a cada 5min) ao scheduler.js|node-cron existente
☐|Adicionar Job 4 (expansão semanal) ao scheduler.js|recurrence.js pronto
☐|Instalar FullCalendar: @fullcalendar/react + plugins|Projeto React existente
☐|Criar src/pages/CalendarPage.jsx (parte 1 — estrutura)|FullCalendar instalado
☐|Criar modal de criação/edição de eventos|CalendarPage base
☐|Criar painel lateral de eventos do dia|CalendarPage base
☐|Criar src/styles/calendar.css e importar no CalendarPage|FullCalendar renderizando
☐|Adicionar rota /calendario no App.jsx|CalendarPage pronto
☐|Adicionar item Calendário ao array NAV da Sidebar|Rota criada
☐|Testar: criar tarefa única → aparece no calendário → notificação chega|Tudo acima
☐|Testar: criar recorrente → múltiplos eventos gerados → notificações|Jobs do scheduler
☐|Testar: marcar evento como feito → some visualmente (opacidade 50%)|PATCH /done
💡  Deploy na Vercel: git push → deploy automático. Cada push no main publica uma nova versão em segundos. Use branches para testar antes de publicar.
:-
```
Itaflix Dashboard — Design System + Wireframes + Código + Prompts  •  Complemento dos arquivos MVP v2.0 e Vibe Coding
Itaflix Dashboard — Guia Completo  |  Página  de
