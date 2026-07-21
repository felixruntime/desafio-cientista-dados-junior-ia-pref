# Spec — Parte 2: Auditoria do Modelo A (Q2 + Q3)

**Branch:** `feat/auditoria-modelo-a`  
**Notebook:** `notebooks/02_auditoria_modelo_a.ipynb`  
**Referência:** [README — Parte 2](../../README.md#parte-2-auditoria-do-modelo-em-produção)

---

## Objetivo

Auditar o **modelo A** (em produção): quantificar desempenho global e por categoria **com incerteza**, identificar subgrupos e modos de falha, e discutir impacto prático no encaminhamento dos chamados.

---

## Entregáveis

### Q2 — Desempenho com incerteza

- [ ] Métricas globais com intervalos de confiança (95%)
- [ ] Métricas por categoria com intervalos de confiança
- [ ] Tabela consolidada de resultados
- [ ] Método de cálculo dos IC explicitado no notebook
- [ ] Justificativa da escolha das métricas (desbalanceamento)

### Q3 — Onde o modelo falha?

- [ ] Matriz de confusão (absoluta e/ou normalizada por linha)
- [ ] Principais pares de confusão quantificados
- [ ] Análise por subgrupos (canal, quartil de texto, etc.)
- [ ] Hipóteses sobre causas dos erros
- [ ] Discussão do **impacto prático** de cada modo de falha

---

## Decisões metodológicas

| Decisão | Escolha proposta | Justificativa |
|---|---|---|
| Métrica global | Acurácia + macro-F1 | Acurácia é interpretável; macro-F1 trata classes igualmente |
| Por categoria | Recall, precision, F1 | Desbalanceamento exige olhar classe a classe |
| Intervalos de confiança | Bootstrap estratificado (B=1000, 95%) | Flexível; funciona para métricas e subgrupos |
| Matriz de confusão | Normalizada por linha (recall) | Mostra para onde vão os erros de cada classe |
| Subgrupos | Canal, quartil de comprimento do texto | Hipóteses levantadas na EDA |
| Calibração | Confiança vs acurácia (`conf_modelo_a`) | Modelo A declara confiança — avaliar utilidade |

---

## Análises obrigatórias

### Q2 — Métricas e incerteza

1. **Global**
   - Acurácia: `pred_modelo_a == categoria_real`
   - Macro-F1 (média não ponderada do F1 por classe)
   - IC 95% via bootstrap

2. **Por categoria**
   - Recall, precision, F1 para cada uma das 8 categorias
   - IC 95% via bootstrap estratificado por `categoria_real`

3. **Documentar no notebook**
   - Por que acurácia sozinha é insuficiente
   - Como o bootstrap foi implementado (n resamples, seed, nível de confiança)

### Q3 — Modos de falha

1. **Matriz de confusão**
   - Heatmap: linhas = categoria real, colunas = predição A
   - Top 5–10 pares de confusão (real → predito) com contagem e %

2. **Subgrupos**
   - Acurácia (ou recall) por `canal`
   - Acurácia por quartil de comprimento do texto
   - (Opcional) Por bairro, se houver sinal na EDA

3. **Confiança do modelo A**
   - Confiança média quando acerta vs quando erra
   - (Opcional) Curva confiança vs acurácia ou taxa de erro em faixas de confiança

4. **Hipóteses e impacto**
   - Para cada modo de falha principal:
     - **Quantificação** (quantos casos, % da classe)
     - **Hipótese de causa** (ex.: esgoto confundido com buraco — categorias semanticamente próximas)
     - **Impacto prático** (ex.: encaminhamento errado atrasa reparo de infraestrutura)

---

## Visualizações esperadas

| Visualização | Propósito |
|---|---|
| Tabela — métricas globais com IC | Q2 |
| Tabela — métricas por categoria com IC | Q2 |
| Heatmap — matriz de confusão | Q3 |
| Barplot — top confusões | Q3 |
| Barplot — acurácia por subgrupo | Q3 |
| (Opcional) Confiança vs acurácia | Calibração |

---

## Definition of Done

- [ ] Notebook executa sem erro do início ao fim
- [ ] IC calculados e método documentado
- [ ] Justificativa das métricas presente
- [ ] Modos de falha quantificados (não só descritos)
- [ ] Impacto prático discutido para falhas principais
- [ ] Conexão explícita com achados da EDA (Parte 1)
- [ ] Branch mergeada na `main`

---

## Riscos e limitações a documentar

- Bootstrap assume amostra representativa — dados sintéticos
- IC por categoria podem ser largos em classes pequenas (ex.: `sinalizacao`)
- Subgrupos com n baixo — interpretar com cautela
- Auditoria do A não implica que B seja melhor — isso é Parte 3

---

## Conexão com outras partes

| Parte | Conexão |
|---|---|
| Parte 1 (EDA) | Subgrupos e hipóteses vêm da EDA |
| Parte 3 (A vs B) | Falhas do A (ex.: esgoto→buraco) servem de baseline para comparar B |
