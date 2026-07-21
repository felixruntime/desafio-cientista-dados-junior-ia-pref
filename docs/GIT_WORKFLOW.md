# Git Workflow

Convenções de branches, commits e merge para este repositório.

## Visão geral

Este projeto segue entregas incrementais alinhadas às partes do desafio. A branch `main` permanece estável; cada parte analítica é desenvolvida em branch dedicada e mergeada ao concluir.

Pull requests são **opcionais**. O histórico de commits e a organização do repositório são o foco principal.

## Branches

| Branch | Escopo | Merge |
|---|---|---|
| `chore/setup` | Estrutura, specs, `requirements.txt` | Direto na `main` |
| `feat/eda` | Notebook 01 — Análise exploratória (Q1) | Direto na `main` |
| `feat/auditoria-modelo-a` | Notebook 02 — Auditoria do modelo A (Q2 + Q3) | Direto na `main` |
| `feat/comparacao-recomendacao` | Notebook 03 — A vs B + sumário executivo (Q4) | Direto na `main` |
| `feat/bonus-llm` | Bônus — Classificação com LLM (opcional) | Direto na `main` |

### Convenção de nomenclatura

```
<tipo>/<escopo-curto>
```

**Tipos permitidos:**

| Tipo | Uso |
|---|---|
| `chore` | Setup, configuração, estrutura |
| `feat` | Nova análise ou entrega |
| `fix` | Correção de bug ou erro em notebook |
| `docs` | Documentação isolada (README, specs) |

**Exemplos:** `feat/eda`, `feat/auditoria-modelo-a`, `chore/setup`

## Commits

Seguimos [Conventional Commits](https://www.conventionalcommits.org/) simplificado:

```
<tipo>(<escopo>): <descrição no imperativo>
```

### Escopos sugeridos

| Escopo | Uso |
|---|---|
| `setup` | Estrutura e dependências |
| `eda` | Parte 1 — Análise exploratória |
| `audit` | Parte 2 — Auditoria do modelo A |
| `compare` | Parte 3 — Comparação e recomendação |
| `readme` | README e sumário executivo |
| `spec` | Especificações em `docs/specs/` |

### Exemplos

```
chore(setup): add project directory structure
docs(spec): add EDA specification
feat(eda): plot category distribution and class imbalance
feat(eda): analyze text length patterns by channel
feat(audit): compute model A metrics with bootstrap CIs
feat(audit): add confusion matrix and failure mode analysis
feat(compare): run McNemar test for paired model comparison
docs(readme): add executive summary and reproduction steps
```

### Regras

- Um propósito lógico por commit
- Descrição no imperativo ("add", "fix", "analyze")
- Evitar commits genéricos ("updates", "fix", "wip")
- Não commitar outputs pesados desnecessários (figures são regeneráveis)

## Fluxo de trabalho

```text
main
 │
 ├── chore/setup ──merge──► main
 │
 ├── feat/eda ──merge──► main
 │
 ├── feat/auditoria-modelo-a ──merge──► main
 │
 └── feat/comparacao-recomendacao ──merge──► main
```

### Passo a passo por entrega

1. Atualizar a spec correspondente em `docs/specs/` (se necessário)
2. Criar branch a partir da `main` atualizada
3. Desenvolver com commits incrementais
4. Verificar que notebooks rodam do início ao fim
5. Merge na `main` (local ou via PR no fork, se preferir)

### Merge

```bash
git checkout main
git merge --no-ff feat/eda -m "merge: feat/eda into main"
git push origin main
```

`--no-ff` preserva o ponto de merge no histórico (opcional, mas recomendado).

## Specs analíticas

Cada parte do desafio possui spec em `docs/specs/`. Consulte antes de iniciar a branch correspondente:

- [01 — EDA](specs/01-eda.md)
- [02 — Auditoria Modelo A](specs/02-auditoria-modelo-a.md)
- [03 — Comparação e Recomendação](specs/03-comparacao-recomendacao.md)

Índice completo: [SPECS.md](SPECS.md)

## Referências

- [README.md](../README.md) — enunciado oficial do desafio
- Specs em `docs/specs/` — plano e checklist por entrega
