# Submetendo o Infleux for Brands como MCP Connector

Guia para inscrever o servidor MCP da Infleux no diretório de Connectors da Anthropic
(claude.ai, Claude Desktop e Claude Code). É uma submissão separada da do plugin — ver
[SUBMISSAO-MARKETPLACE.md](SUBMISSAO-MARKETPLACE.md).

Os textos em bloco de citação estão em inglês porque o diretório é global — copie como estão.

---

## 0. Pendências antes de enviar

- [x] Conta de revisão + marca demo + fixtures densos (seção 6) — `demo@infleux.co`
- [ ] Conferir as anotações das tools no código do servidor (seção 7)
- [ ] Testar o login pelo claude.ai como custom connector (seção 7)
- [ ] Confirmar se `pre_campaigns:write` vem por padrão em contas de marca (seção 4)
- [ ] Decidir o e-mail de suporte: `suporte@infleux.co` ou `hello@infleux.co`
- [ ] Avaliar uma política de privacidade que cubra anunciantes/marcas (seção 1)

---

## 1. Identificação

| Campo | Valor |
| --- | --- |
| Nome | Infleux for Brands |
| Operador | Infleux |
| URL do servidor | `https://mcp.infleux.io/mcp` |
| Transporte | Streamable HTTP (remoto) |
| Site | https://www.infleux.co |
| Documentação | https://github.com/revmobgithub/infleux-brands-mcp-plugin |
| Política de privacidade | https://legal.infleux.co/pp-influencer/2023-02-17/20230217_politica-de-privacidade-v8.pdf |
| Suporte | suporte@infleux.co (manifesto público usa hello@infleux.co — alinhar) |
| Categoria | Marketing / Productivity |
| Logo | `plugins/infleux-brands/assets/icon-512.png` |
| Tipo de acesso | **Read-write** |

> A política atual é a de influenciadores (`pp-influencer`, 2023, em português). A revisão
> pode questionar se ela cobre os dados de marcas e campanhas que o conector acessa. Se
> existir uma política para anunciantes, ou uma versão em inglês, prefira essa.

---

## 2. Autenticação

Verificado contra produção em 2026-10-02.

- **Tipo:** OAuth 2.1 + PKCE (`S256`). O servidor responde `401` com
  `WWW-Authenticate: Bearer resource_metadata=...`.
- **Protected resource metadata:** `https://mcp.infleux.io/.well-known/oauth-protected-resource`
- **Authorization server:** `https://auth.infleux.io` (Zitadel) —
  discovery em `/.well-known/openid-configuration`
- **Dynamic Client Registration:** suportado (`https://auth.infleux.io/oauth/v2/register`)
  — não há client ID/secret para informar.
- **Scopes:** `openid profile email` + audience/roles do projeto Zitadel
- **Refresh token:** suportado
- **Callbacks do Claude** que o Zitadel precisa aceitar:
  `https://claude.ai/api/mcp/auth_callback` e `https://claude.com/api/mcp/auth_callback`

---

## 3. Descrições

### Curta

> Run your Infleux influencer marketing campaigns from Claude — live campaigns, performance, approvals and drafts.

### Longa

> Infleux for Brands connects Claude to the Infleux influencer marketing platform, so brands and agencies can work with live campaign data in conversation instead of clicking through dashboards. Ask which campaigns are running and what they pay, pull clicks, conversions and spend for any period, inspect where traffic comes from, check what is waiting on your approval, and draft the next edition of a campaign. Authentication is OAuth with your existing Infleux account; access is scoped server-side to your Infleux role. Every write requires explicit confirmation after a preview; everything else is read-only. Requires an Infleux account.

### Tipo de acesso (observação)

> Read-write. 28 of 30 tools are read-only. The only two write tools create and edit pre-campaign drafts (unpublished). Every write returns a preview first and only executes after explicit user confirmation (`confirmed: true`). No tool deletes data, publishes a campaign, or moves money.

### Dados e privacidade

> A brand account reads only its own campaigns, castings, approval queues, actions, budgets, spend and click analytics, and can draft pre-campaigns. Influencer personal data and creator earnings are blocked server-side by scope. No raw SQL — analytics queries are built server-side from fixed templates. Operated by Infleux at mcp.infleux.io; identity at auth.infleux.io.

---

## 4. Casos de uso, prompts e pré-requisitos

Cada caso tem o **mesmo prompt em inglês e em português** (use qualquer idioma no Claude).
Nomes de campanha abaixo batem com a conta demo da seção 6.

### Casos de uso + prompts de sugestão

**1. Listar campanhas recentes / ativas**

> List the brand’s latest or currently live campaigns (name, window, payout model).

- EN: `What are the latest campaigns created?`
- PT: `Quais são as últimas campanhas criadas?`
- EN: `Which campaigns is Demo Brand running right now?`
- PT: `Quais campanhas a Demo Brand está rodando agora?`

**2. Campaign performance review**

> Pull clicks, conversions and spend for any campaign and period, read as a funnel (views → creators running → clicks → conversions → spend) so a drop can be traced to where it started.

- EN: `How did "Demo Brand — Spring Launch Oct/26" perform in the last 30 days? Show clicks, conversions and spend, and tell me where the funnel dropped.`
- PT: `Como foi o desempenho de "Demo Brand — Spring Launch Oct/26" nos últimos 30 dias? Mostre cliques, conversões e gasto, e diga onde o funil caiu.`

**3. Top converting creators across campaigns**

> Rank influencers/actions by conversions (or clicks) for one or more named campaigns.

- EN: `Which influencers converted the most on the campaigns "Demo Brand — Spring Launch Oct/26" and "Demo Brand — Evergreen CPA Sep/26"?`
- PT: `Quais foram os influenciadores que converteram mais nas campanhas "Demo Brand — Spring Launch Oct/26" e "Demo Brand — Evergreen CPA Sep/26"?`

**4. Approval triage**

> See everything waiting on the brand — creators awaiting approval and content awaiting review — grouped by campaign and ordered by age, so nothing stalls in the queue.

- EN: `What's waiting on our approval this week? Group it by campaign, oldest first.`
- PT: `O que está esperando nossa aprovação esta semana? Agrupe por campanha, do mais antigo para o mais novo.`

**5. Pending pre-campaigns (pipeline)**

> List unpublished drafts still awaiting approval.

- EN: `Which pre-campaigns are still pending approval for Demo Brand?`
- PT: `Quais pré-campanhas da Demo Brand ainda estão pendentes de aprovação?`

**6. Briefing / rules**

> Open briefing, must-do / must-not-do and payouts for a named campaign.

- EN: `Show me the briefing and payout rules for "Demo Brand — Spring Launch Oct/26".`
- PT: `Mostre o briefing e as regras de pagamento de "Demo Brand — Spring Launch Oct/26".`

**7. Relaunching a campaign**

> Clone a published campaign into a new draft, adjust dates, briefing and payouts, and review a full preview before anything is saved.

- EN: `Clone "Demo Brand — Evergreen CPA Sep/26" as a draft for next month and show me the preview before saving.`
- PT: `Clone "Demo Brand — Evergreen CPA Sep/26" como rascunho para o mês que vem e me mostre o preview antes de salvar.`

### Pré-requisitos

> - An active **Infleux platform account** with a brand / advertiser (or agency) profile. Accounts are provisioned by Infleux; there is no self-serve sign-up — contact your Infleux account manager or hello@infleux.co.
> - The account must be linked to at least one brand in Infleux. Data access is scoped server-side to the brands and permissions assigned to that account's role.
> - Write actions (pre-campaign drafts) require the `pre_campaigns:write` permission on the account; without it the connector works read-only.
> - No API keys or configuration: users connect by signing in with their Infleux credentials via OAuth.
> - Influencer personal data and creator earnings are restricted to Infleux staff accounts and are not available to brand accounts.

Se toda conta de marca já recebe `pre_campaigns:write`, remova o terceiro item.

---

## 5. Tools (30)

**Leitura (28, das quais 8 só para staff):**

| Grupo | Tools |
| --- | --- |
| Sessão | `ping_core`, `whoami`, `get_query_reference` |
| Marcas | `find_brands`, `find_advertisers`, `find_advertiser_brands` |
| Campanhas | `find_campaigns`, `find_live_campaigns_for_brand`, `get_campaign`, `get_campaign_casting`, `find_campaign_queue`, `find_content_approval_queue`, `find_campaign_views`, `find_campaigns_available`, `get_campaign_available` |
| Performance | `find_actions`, `get_action`, `query_athena_clicks`, `query_action_tracker_events`, `query_athena_spend`, `find_budgets` |
| Rascunhos | `find_pre_campaigns`, `get_pre_campaign` |
| Só staff (403 para marcas) | `find_influencers`, `query_spine_influencers`, `get_influencer`, `find_influencer_events`, `find_influencer_tags`, `query_influencer_earnings`, `find_automated_castings`, `get_automated_casting` |

**Escrita (2)** — preview + `confirmed: true`, nada é apagado:
`clone_campaign_to_pre_campaign`, `update_pre_campaign`.

**Extras:** resources `infleux://about`, `infleux://glossary`, `infleux://spine-select` e
7 prompts (`live_campaigns_for_brand`, `campaign_rules_and_briefing`,
`campaign_queue_for_campaign`, `content_approval_queue_for_campaign`, `campaign_views`,
`clone_campaign`, `edit_pre_campaign`).

---

## 6. Conta de revisão (test setup)

Sem conta de teste o revisor para na tela de login e a submissão atrasa ou é reprovada.

### Credenciais (colar no formulário da Anthropic)

| Campo | Valor |
| --- | --- |
| Email | `demo@infleux.co` |
| Password | `DemoInfleux!2026` |
| Marca | **Infleux Demo Account** / campanhas rotuladas **Demo Brand** |
| MFA | desligado |
| Dados | fictícios (fixtures densos no MCP: ~30 campanhas, filas, actions, pré-campanhas approved + pending) |

Conta dedicada à revisão do connector — **não** é conta de cliente. Depois da aprovação da Anthropic, trocar a senha ou desativar, salvo se pedirem para manter.

### Checklist (estado atual)

- [x] Conta dedicada `demo@infleux.co` (não pessoal / não cliente)
- [x] Marca demo sem dados reais de clientes
- [x] Várias campanhas ativas + encerradas + upcoming, com briefing e payouts
- [x] Cliques / conversões / spend mockados para review de performance
- [x] Filas de aprovação de criadores **e** de conteúdo
- [x] Campanha publicada clonável: `Demo Brand — Evergreen CPA Sep/26`
- [x] Pré-campanhas: 30 `approved` (ligadas às campanhas) + 8 `pending`
- [x] Sem MFA; senha estável para a janela de review
- [ ] Teste de ponta a ponta no claude.ai (custom connector → login → fluxo abaixo)
- [ ] Pós-aprovação: rotacionar senha se a Anthropic não pedir para manter

### Texto do campo "Test setup instructions"

Copie o bloco abaixo para o formulário (inglês — idioma do diretório). Os prompts em português equivalentes estão na seção 4.

> **Test account**
> Email: `demo@infleux.co`
> Password: `DemoInfleux!2026`
> (Dedicated reviewer account on the Infleux Demo Account / Demo Brand sample data — no real customer data. No MFA.)
>
> **Setup**
> 1. Add the connector with the URL `https://mcp.infleux.io/mcp`.
> 2. Click Connect. You will be redirected to auth.infleux.io; sign in with the credentials above and approve access.
> 3. No other configuration is needed.
>
> **Suggested test flow** (English; Portuguese equivalents in our submission notes §4)
> 1. `Who am I signed in as in Infleux?` — confirms the connection (expect Demo Brand / Infleux Demo Account).
> 2. `What are the latest campaigns created?` — lists recent Demo Brand campaigns (sample pack has ~30).
> 3. `Which campaigns is Demo Brand running right now?` — live campaigns only.
> 4. `How did "Demo Brand — Spring Launch Oct/26" perform in the last 30 days?` — clicks, conversions and spend.
> 5. `Which influencers converted the most on the campaigns "Demo Brand — Spring Launch Oct/26" and "Demo Brand — Evergreen CPA Sep/26"?` — ranks creators/actions by conversions.
> 6. `What's waiting on our approval?` — creator and content queues.
> 7. `Which pre-campaigns are still pending approval for Demo Brand?` — unpublished pipeline.
> 8. `Clone "Demo Brand — Evergreen CPA Sep/26" as a draft for next month.` — the connector returns a preview and asks for confirmation; nothing is written until you confirm. Confirming creates an unpublished draft only.
>
> **Expected permission boundary**
> Asking for influencer PII profiles or creator earnings (e.g. `Show me influencer earnings`) returns a 403 "Missing scope" — this is intentional: those tools are restricted to Infleux staff accounts. Brand-scoped action/conversion rankings (step 5) are allowed.
>
> **Notes**
> The account and its sample data will remain available throughout the review. Contact: suporte@infleux.co.

---

## 7. Verificações técnicas antes de enviar

1. **Anotações das tools.** O diretório exige `title` e `readOnlyHint`/`destructiveHint` em
   todas as tools — motivo comum de reprovação. Leitura: `readOnlyHint: true`. As duas de
   escrita: `readOnlyHint: false`, `destructiveHint: false`. Conferir no código do servidor.
2. **Login pelo claude.ai.** No Claude Code o callback é local; no claude.ai é
   `https://claude.ai/api/mcp/auth_callback`. Confirmar que o cliente criado por DCR no
   Zitadel aceita esse redirect, adicionando o conector como custom connector.
3. **Endpoint como contrato público.** Após a aprovação, mudanças de URL, autenticação ou
   remoção de tool em `https://mcp.infleux.io/mcp` quebram todos os usuários conectados.
