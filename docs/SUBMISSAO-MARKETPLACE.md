# Publicando o plugin no diretório de plugins da Claude

Guia operacional para levar o `infleux-brands` do repositório local até o diretório
oficial de plugins da Anthropic. Três etapas: validar, publicar o repositório, submeter.

---

## 0. Entenda os dois caminhos de distribuição

| Caminho | O que é | Quando usar |
| --- | --- | --- |
| **Marketplace próprio** (este repo) | Parceiro roda `/plugin marketplace add revmobgithub/infleux-brands-mcp-plugin` e instala | Disponível **hoje**, sem depender de aprovação. Use para o onboarding dos parceiros desde já. |
| **Diretório oficial da Anthropic** | O plugin aparece na busca nativa do Claude, sem o parceiro adicionar marketplace nenhum | Alcance máximo. Passa por revisão de qualidade e segurança. |

Os dois convivem: o repositório é a fonte, e o diretório oficial aponta para ele. Comece
pelo primeiro e submeta ao segundo em paralelo.

---

## 1. Validar antes de publicar

```bash
claude plugin validate .
```

Valida `marketplace.json` e `plugin.json` contra o schema. Precisa passar sem erros.

Teste a instalação a partir do checkout local, numa sessão interativa do Claude Code:

```
/plugin marketplace add ./
/plugin install infleux-brands@infleux
/mcp
```

Checklist do teste manual — faça de verdade, é o mesmo caminho que o revisor vai seguir:

- [ ] `/mcp` mostra **infleux** e o fluxo OAuth abre no navegador
- [ ] Login com conta Infleux de parceiro (perfil `advertiser`, não staff) conclui
- [ ] `/infleux-status` responde com o usuário logado
- [ ] `/infleux-campaigns <marca>` traz campanhas live
- [ ] `/infleux-performance <campanha>` traz cliques/conversões/spend
- [ ] Uma conta de parceiro recebe `Missing scope` nas tools de influenciador — e o Claude
      reporta isso como limite de permissão, não como erro
- [ ] `update_pre_campaign` sem `confirmed` devolve preview e **não** grava

Se algo falhar aqui, falha na revisão.

---

## 2. Publicar o repositório no GitHub

O diretório oficial busca o plugin direto de um repositório Git público. Ele precisa
existir e estar acessível antes da submissão.

```bash
git init
git add .
git commit -m "feat: infleux-brands plugin for Claude Code"
git branch -M main
git remote add origin git@github.com:revmobgithub/infleux-brands-mcp-plugin.git
git push -u origin main
```

Depois do push:

1. Crie uma **release/tag** (`v1.0.0`). O diretório fixa a versão por `ref` + `sha`, e uma
   tag deixa explícito o que foi revisado.
2. Confirme que o repositório é **público**.
3. Ajuste `repository` e `homepage` no `plugin.json` e no `marketplace.json` se o
   caminho `revmobgithub/infleux-brands-mcp-plugin` mudar. Eles precisam bater com a URL real.

Com isso pronto, qualquer parceiro já consegue instalar:

```
/plugin marketplace add revmobgithub/infleux-brands-mcp-plugin
/plugin install infleux-brands@infleux
```

---

## 3. Submeter ao diretório oficial

Formulário: **https://clau.de/plugin-directory-submission**

Não existe pull request para os repositórios `anthropics/claude-plugins-official` e
`anthropics/claude-plugins-community` — PRs abertos direto neles são fechados
automaticamente. Tudo entra pelo formulário e passa por revisão interna e varredura
automática de segurança.

Dados a informar (todos já resolvidos neste repositório):

| Campo | Valor |
| --- | --- |
| Nome do plugin | `infleux-brands` |
| Nome de exibição | Infleux for Brands |
| Repositório | `https://github.com/revmobgithub/infleux-brands-mcp-plugin` |
| Caminho do plugin no repo | `plugins/infleux-brands` |
| Categoria | `productivity` |
| Homepage | `https://www.infleux.co` |
| Licença | MIT |
| Contato | suporte@infleux.co |
| Ícone | `plugins/infleux-brands/assets/icon-512.png` |

O diretório vai referenciar o plugin como um `git-subdir` — repositório + caminho +
`ref`/`sha` —, que é o padrão de quem publica plugin dentro de um monorepo:

```json
{
  "name": "infleux-brands",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/revmobgithub/infleux-brands-mcp-plugin.git",
    "path": "plugins/infleux-brands",
    "ref": "v1.0.0"
  }
}
```

---

## 3.1 Descrição e casos de uso (texto pronto para o formulário)

O formulário pede descrição e recursos principais. Os blocos abaixo estão em inglês
porque o diretório é global — copie como estão. Eles batem com o `description` do
`plugin.json` e com o README; se alterar um, alinhe os três.

### Descrição curta (uma linha, ~120 caracteres)

> Run your Infleux influencer marketing campaigns from Claude — live campaigns,
> performance, approvals and drafts.

### Descrição (campo principal do formulário)

> **Infleux for Brands** connects Claude to the [Infleux](https://www.infleux.co)
> influencer marketing platform, so brands and their agencies can work with live campaign
> data in conversation instead of clicking through dashboards.
>
> Ask which campaigns are running and what they pay, pull a performance read-out with
> clicks, conversions and spend for any period, inspect where traffic is coming from,
> check what is waiting on your approval, and draft the next edition of a campaign — all
> against the same live data the Infleux dashboard shows.
>
> Authentication is OAuth 2.0 with your existing Infleux account: no tokens to copy, no
> config to edit. What you can see is scoped server-side to your Infleux role. Every write
> requires an explicit confirmation after a preview; everything else is read-only.
>
> Requires an Infleux account. Talk to your Infleux account manager if you do not have one.

### Principais recursos

> - **Campaign visibility** — live campaigns per brand with window, payout and conversion
>   model; full briefing and rules for any campaign.
> - **Performance analysis** — clicks, conversions and spend for a period, read as a funnel
>   (views → creators running → clicks → conversions → spend) so a drop can be traced to
>   where it started.
> - **Traffic quality** — click-origin breakdown by city, IP, user agent and day, to review
>   suspicious patterns; geo ranking of the creators driving traffic from a region.
> - **Tracking diagnostics** — per-link clicks, conversions and skipped conversions, for
>   confirming a campaign link fires end to end.
> - **Budget and spend** — spend by brand, campaign or period, against the monthly budget
>   it runs on.
> - **Approval queues** — creators awaiting brand review and content awaiting review,
>   grouped by campaign and ordered by age (read-only; decisions stay in the dashboard).
> - **Pre-campaign drafting** — clone a published campaign into a new draft, edit briefing,
>   payouts and content rules, always behind a preview-then-confirm gate.
> - **Four commands** — `/infleux-status`, `/infleux-campaigns`, `/infleux-performance`,
>   `/infleux-pending` — plus three skills Claude loads on its own and a read-only analyst
>   agent for multi-step questions.
> - **No executables** — the plugin is a manifest, an MCP server URL and Markdown. No
>   scripts, no hooks, no bundled binaries.

### Casos de uso

Exemplos concretos, com o que o plugin faz em cada um. Bons para o campo de casos de uso
do formulário e para o material de onboarding dos parceiros.

> **1. Morning check on what is live**
> *"Which campaigns is Acme running right now, and what do they pay?"*
> Resolves the brand, lists live campaigns with window, payout and conversion type, and
> flags when more results exist than were shown.
>
> **2. Monthly performance review**
> *"How did the September campaign perform — clicks, conversions and spend?"*
> Reads the campaign's conversion model, pulls active creators, click analytics and spend
> for the period, and returns a headline, a metrics table, and where in the funnel the
> movement started.
>
> **3. Traffic quality review**
> *"Where are the clicks on this campaign coming from? Anything that looks off?"*
> Breaks clicks down by city, IP, user agent and day, and flags concentration patterns as
> signals to review — never as a fraud verdict.
>
> **4. Tracking debug before a launch**
> *"I clicked the test link an hour ago — did the conversion land?"*
> Aggregates clicks, conversions and skipped conversions for that one tracked link, so a
> broken postback is separated from a creator who is not driving traffic.
>
> **5. Weekly approval triage**
> *"What is waiting on us this week?"*
> Lists creators awaiting brand approval and content awaiting review, grouped by campaign,
> oldest first, plus drafts still pending launch.
>
> **6. Relaunching a campaign**
> *"Run last month's campaign again in October."*
> Clones the published campaign into a pending pre-campaign, asks for the new start and end
> dates (the clone never copies dates), shows the full preview, and only writes after an
> explicit yes.
>
> **7. Budget tracking**
> *"How much has this brand spent this month, and against what budget?"*
> Resolves the advertiser/brand pair, queries spend for the period, and puts it next to the
> monthly budget it runs against.

### Público-alvo e pré-requisitos

> **Who it is for:** brands, advertisers and agencies running influencer campaigns on
> Infleux, plus the Infleux team.
> **Requirements:** an Infleux platform account. The plugin installs without one, but every
> tool call is authenticated and authorized server-side — without an account there is no
> data access.

### Privacidade e dados (se o formulário perguntar)

> Data stays within the authenticated user's own permissions. A brand account reads its
> campaigns, castings, approval queues, actions, budgets, spend and click analytics, and
> can draft pre-campaigns. Influencer profile data and creator earnings are not accessible
> to brand accounts — those tools are rejected server-side by scope. Credentials are held
> by Claude Code's secure storage via OAuth; nothing is stored in the plugin repository.

---

## 4. O que a revisão olha

A Anthropic avalia plugins externos por qualidade e segurança. O que costuma reprovar,
e como este plugin se posiciona:

| Critério | Situação |
| --- | --- |
| Código executável não auditável | Nenhum — só manifesto, URL do MCP e Markdown |
| Segredos no repositório | Nenhum — autenticação é OAuth por usuário |
| Descrição vaga ou inflada | Descrição concreta, com o que o plugin realmente faz |
| Ações destrutivas sem confirmação | Escritas exigem `confirmed: true` após preview |
| Escopo de permissão excessivo | Autorização por papel, aplicada no servidor |
| Documentação ausente | README no plugin e no repo, com pré-requisitos e suporte |
| Nome imitando marketplace oficial | Marketplace chama-se `infleux` |

Pontos que provavelmente vão perguntar — tenha a resposta pronta:

- **O plugin exige conta Infleux.** Deixe claro na submissão que é um conector de produto,
  não uma ferramenta genérica: sem conta, o parceiro instala mas não acessa dado nenhum.
- **Qual dado sai da plataforma.** Dados de campanha, performance e aprovação da própria
  marca do usuário autenticado; nada de dados pessoais de influenciador para contas de
  marca (bloqueado por escopo, no servidor).
- **Quem opera o servidor MCP.** Infleux, em `mcp.infleux.io`, com identidade em
  `auth.infleux.io`.

---

## 5. Depois de aprovado

- **Atualizações** exigem nova submissão ou re-apontamento de `ref`/`sha`, conforme a
  Anthropic instruir na aprovação. Suba `version` no `plugin.json` e no
  `marketplace.json` juntos, e registre no `CHANGELOG.md`.
- **Renomear ou remover** um plugin depois de publicado quebra quem já instalou. Use o
  campo `renames` no `marketplace.json` para migrar sem erro de `plugin-not-found`.
- **O endpoint do MCP vira contrato público.** Mudança em `https://mcp.infleux.io/mcp`
  — URL, autenticação, remoção de tool — quebra todos os parceiros instalados. Trate como
  API pública versionada.

---

## Referências

- [Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Create and distribute a plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)
- [anthropics/claude-plugins-community](https://github.com/anthropics/claude-plugins-community)
- [Formulário de submissão](https://clau.de/plugin-directory-submission)
