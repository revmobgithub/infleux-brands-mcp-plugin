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
