# Spec — Parte 1: Análise Exploratória (Q1)

**Branch:** `feat/eda`  
**Notebook:** `notebooks/01_analise_exploratoria.ipynb`  
**Referência:** [README — Parte 1](../../README.md#parte-1-análise-exploratória)

---

## Objetivo

Explorar o corpus de chamados e identificar padrões relevantes para o problema de **classificação automática**. A EDA deve informar a auditoria dos modelos (Parte 2) e a comparação A vs B (Parte 3) — não é uma exploração genérica de dados.

---

## Entregáveis

- [ ] Notebook documentado que roda do início ao fim sem erros
- [ ] Visualizações e tabelas focadas no problema de classificação
- [ ] **Síntese de 3–5 achados** com interpretação em texto
- [ ] Para cada achado: **por que importa** para avaliar os classificadores
- [ ] Figuras salvas em `results/figures/` (quando aplicável)

---

## Decisões metodológicas

| Decisão | Escolha proposta | Justificativa |
|---|---|---|
| Dataset | `dados/chamados_com_predicoes.csv` | 5.000 chamados rotulados |
| Foco | Variáveis ligadas ao texto e à classificação | Enunciado pede foco no problema de classificação |
| Métricas descritivas | Contagens, proporções, mediana de caracteres | Base para subgrupos na Parte 2 |
| Visualizações | matplotlib / seaborn | Reprodutibilidade e simplicidade |

---

## Análises obrigatórias

### 1. Panorama geral

- Dimensões do dataset (linhas, colunas, tipos)
- Valores ausentes e duplicatas
- Período temporal (`data_abertura`)

### 2. Distribuição de categorias

- Contagem e proporção por `categoria_real`
- Identificar **desbalanceamento** entre classes
- Implicação: acurácia global pode mascarar desempenho em classes minoritárias

### 3. Características dos textos

- Distribuição do comprimento (`len(texto)`)
- Comprimento por categoria
- Exemplos de textos curtos vs longos
- Implicação: textos curtos podem ser mais difíceis de classificar

### 4. Padrões por canal

- Distribuição de chamados por `canal` (app, telefone, portal)
- Categoria × canal (tabela ou heatmap)
- Comprimento médio do texto por canal
- Implicação: canal pode ser proxy de qualidade/completude do texto

### 5. Padrões por bairro e tempo (se relevante)

- Distribuição por `bairro`
- Volume ao longo do tempo (opcional)
- Focar apenas se houver sinal — evitar EDA por EDA

---

## Visualizações esperadas

| Visualização | Propósito |
|---|---|
| Barplot — categorias | Desbalanceamento de classes |
| Histograma / boxplot — comprimento do texto | Variabilidade do input dos modelos |
| Heatmap ou stacked bar — categoria × canal | Padrões de entrada por canal |
| (Opcional) Série temporal de volume | Sazonalidade ou tendência |

---

## Síntese final (template)

Ao final do notebook, responder explicitamente:

1. **Achado 1:** [descrição] → **Relevância para classificação:** [por quê]
2. **Achado 2:** ...
3. **Achado 3:** ...
4. (Opcional) Achados 4 e 5

---

## Definition of Done

- [ ] Notebook executa sem erro do início ao fim
- [ ] 3–5 achados interpretados (não só gráficos)
- [ ] Achados conectados ao problema de classificação
- [ ] Código com células markdown explicativas
- [ ] Branch `feat/eda` mergeada na `main`

---

## Riscos e limitações a documentar

- Dados sintéticos — padrões podem não refletir produção real
- Bairros/canais com poucas observações — interpretar com cautela
- EDA não substitui avaliação dos modelos — prepara hipóteses para Parte 2

---

## Hipóteses para validar na Parte 2

Registrar aqui após a EDA (preencher no notebook):

- [ ] Classes desbalanceadas afetam recall por categoria?
- [ ] Textos curtos têm pior desempenho?
- [ ] Algum canal apresenta padrão diferente?
- [ ] Categorias semanticamente próximas existem no corpus?
