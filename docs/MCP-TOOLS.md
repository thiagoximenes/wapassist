# MCP Tools — Ferramentas com Model Context Protocol

> **Regra permanente para o agente de IA:** Sempre que uma ação envolver as ferramentas listadas abaixo, **use o MCP correspondente** em vez de gerar código ou comandos manuais. Isso garante execução direta, sem erros de digitação e com feedback imediato.

---

## MCP 1 — Vercel (`vercel`)

**Quando usar:** Qualquer ação relacionada ao deploy e operação do frontend (`wapassist-dashboard`).

| Situação | Ação via MCP |
|---|---|
| Fazer deploy do frontend | `mcp2_deploy_to_vercel` |
| Verificar status de um deploy | `mcp2_get_deployment` |
| Ver logs de build com erro | `mcp2_get_deployment_build_logs` |
| Ver logs de runtime (erros em produção) | `mcp2_get_runtime_logs` |
| Listar todos os projetos Vercel | `mcp2_list_projects` |
| Listar deploys de um projeto | `mcp2_list_deployments` |
| Acessar URL protegida da Vercel | `mcp2_get_access_to_vercel_url` ou `mcp2_web_fetch_vercel_url` |
| Verificar disponibilidade de domínio | `mcp2_check_domain_availability_and_price` |
| Buscar documentação da Vercel | `mcp2_search_vercel_documentation` |

**Identificadores úteis:**
- Team ID: obtido via `mcp2_list_teams`
- Project ID: obtido via `mcp2_list_projects` ou lendo `.vercel/project.json`

---

## MCP 2 — Supabase (`supabase-mcp-server`)

> ⚠️ O projeto usa **Neon.tech** por padrão. Se em algum momento for decidido migrar para Supabase (mesmo PostgreSQL, painel mais rico, auth nativa), este MCP elimina toda configuração manual.

**Quando usar:** Se o banco for migrado para Supabase, usar este MCP para todas as operações de banco.

| Situação | Ação via MCP |
|---|---|
| Criar projeto Supabase | `mcp1_create_project` |
| Executar SQL direto | `mcp1_execute_sql` |
| Aplicar migration DDL | `mcp1_apply_migration` |
| Listar tabelas | `mcp1_list_tables` |
| Gerar tipos TypeScript | `mcp1_generate_typescript_types` |
| Ver logs da API / banco / auth | `mcp1_get_logs` |
| Verificar advisors de segurança | `mcp1_get_advisors` |
| Deploy de Edge Function | `mcp1_deploy_edge_function` |
| Criar branch de desenvolvimento | `mcp1_create_branch` |
| Obter URL e chaves do projeto | `mcp1_get_project_url` + `mcp1_get_publishable_keys` |

---

## MCP 3 — Playwright (`mcp-playwright`)

**Quando usar:** Testes automatizados de interface da dashboard, validação de fluxos end-to-end e debug visual.

| Situação | Ação via MCP |
|---|---|
| Navegar para uma URL | `mcp0_browser_navigate` |
| Tirar screenshot da tela | `mcp0_browser_take_screenshot` |
| Capturar snapshot de acessibilidade | `mcp0_browser_snapshot` |
| Clicar em elemento | `mcp0_browser_click` |
| Preencher formulário | `mcp0_browser_fill_form` ou `mcp0_browser_type` |
| Ver erros no console do browser | `mcp0_browser_console_messages` |
| Ver requisições de rede | `mcp0_browser_network_requests` |
| Executar JavaScript na página | `mcp0_browser_evaluate` |
| Aguardar elemento aparecer | `mcp0_browser_wait_for` |
| Redimensionar janela | `mcp0_browser_resize` |

**Fluxos de teste recomendados para o wapassist:**

```
1. Login
   → navigate /login → fill senha → click Entrar → wait "Visão Geral"

2. Cadastrar cliente
   → navigate /clientes/novo → fill_form → click Salvar → snapshot confirmação

3. Enviar cobrança manual
   → navigate /clientes/:id → click "Enviar cobrança" → snapshot resultado

4. Verificar dashboard KPIs
   → navigate / → snapshot → avaliar valores dos StatCards
```

---

## Resumo Rápido — Quando Usar Cada MCP

| Preciso... | Usar |
|---|---|
| Fazer deploy ou ver logs do frontend | **Vercel MCP** |
| Operar banco de dados (se Supabase) | **Supabase MCP** |
| Testar a interface da dashboard | **Playwright MCP** |
| Validar um fluxo end-to-end visualmente | **Playwright MCP** |
| Debug de erro em produção no frontend | **Vercel MCP** (`get_runtime_logs`) |
| Verificar se o deploy passou | **Vercel MCP** (`get_deployment_build_logs`) |
