# Spec — Parte 3: Comparação A vs B e Recomendação (Q4)

**Branch:** `feat/comparacao-recomendacao`  
**Notebook:** `notebooks/03_comparacao_e_recomendacao.ipynb`  
**Referência:** [README — Parte 3](../../README.md#parte-3-modelo-a-vs-modelo-b)

---

## Objetivo

Comparar **modelo A** (produção) e **modelo B** (candidato) com teste estatístico adequado ao desenho pareado, analisar trade-offs por categoria, e produzir **recomendação acionável** para gestor não técnico.

---

## Entregáveis

- [ ] Comparação global com teste de hipótese pareado
- [ ] Justificativa da escolha do teste
- [ ] Interpretação correta do p-valor
- [ ] Comparação por categoria com trade-offs explícitos
- [ ] **Parágrafo de recomendação para gestor** (riscos + mitigação)
- [ ] Mesmo parágrafo no **sumário executivo** do `README.md`
- [ ] Atualização do README: abordagem, como reproduzir, sumário executivo (≤ 1 página)

---

## Decisões metodológicas

| Decisão | Escolha proposta | Justificativa |
|---|---|---|
| Desenho | Pareado (mesmos 5.000 chamados) | Predições A e B sobre mesma amostra |
| Teste global | **McNemar** | Compara classificações pareadas; usa discordâncias |
| Discordâncias | b-only (só B acerta) vs a-only (só A acerta) | Estatística do McNemar |
| Métrica por categoria | Acurácia (recall por classe) + diferença B−A | Trade-offs visíveis por serviço |
| IC por categoria | Bootstrap da diferença de acurácia | Incerteza na vantagem por classe |
| Recomendação | Global + por categoria | Enunciado exige verificar se conclusão global se sustenta |

---

## Análises obrigatórias

### 1. Comparação global

1. Acurácia A vs B (com IC bootstrap)
2. Macro-F1 A vs B (opcional, complementar)
3. Tabela pareada:
   - Ambos acertam
   - Só A acerta
   - Só B acerta
   - Ambos erram

4. **Teste de McNemar**
   - H0: P(B acerta e A erra) = P(A acerta e B erra)
   - Reportar: estatística, p-valor, conclusão
   - Justificar por que não usar teste para amostras independentes

### 2. Comparação por categoria

Para cada uma das 8 categorias:

- Acurácia A e B
- Diferença (B − A)
- IC 95% da diferença (bootstrap)
- Interpretação: B melhora, piora ou empata?

Identificar explicitamente:
- Categorias onde B **ganha substancialmente**
- Categorias onde B **piora** (trade-off crítico)
- Implicação para encaminhamento por tipo de serviço

### 3. Confiança dos modelos (complementar)

- Comparar calibração: confiança quando acerta vs erra (A vs B)
- Utilidade de `conf_modelo_b` para triagem/revisão humana

### 4. Recomendação final

Estrutura sugerida do parágrafo para gestor:

1. **Conclusão principal** — trocar, não trocar, ou troca condicional?
2. **Evidência** — uma frase com números-chave (sem jargão)
3. **Riscos** — o que pode piorar com a troca?
4. **Mitigação** — monitoramento, regra híbrida, revisão humana, etc.
5. **Próximo passo** — ação concreta recomendada

---

## Visualizações esperadas

| Visualização | Propósito |
|---|---|
| Barplot — acurácia A vs B (global) | Comparação visual |
| Tabela — outcomes pareados | Input do McNemar |
| Barplot — diferença B−A por categoria | Trade-offs |
| (Opcional) Heatmap A vs B por categoria | Visão consolidada |

---

## README — sumário executivo

Ao concluir esta parte, atualizar `README.md` com:

### Seções obrigatórias

1. **Abordagem** — breve descrição metodológica
2. **Como reproduzir** — ambiente, dependências, ordem dos notebooks
3. **Sumário executivo** (≤ 1 página):
   - Principais achados (EDA, auditoria A, comparação)
   - Recomendação final (mesmo parágrafo do notebook)
   - Riscos e mitigação

---

## Definition of Done

- [ ] McNemar implementado e interpretado corretamente
- [ ] Comparação por categoria com trade-offs discutidos
- [ ] Recomendação escrita para gestor não técnico
- [ ] README atualizado com sumário executivo
- [ ] Notebook executa sem erro do início ao fim
- [ ] Branch mergeada na `main`

---

## Riscos e limitações a documentar

- Conclusão global pode esconder regressões por categoria
- McNemar testa discordâncias — não mede magnitude do ganho
- Dados sintéticos — validação em produção seria necessária antes de deploy
- Recomendação deve ser honesta sobre incertezas

---

## Cenários de recomendação (referência)

| Cenário | Quando | Exemplo de recomendação |
|---|---|---|
| Troca total | B melhor em global e em todas/most categories | Substituir A por B com monitoramento |
| Troca parcial | B melhor global, piora em categorias específicas | Regra híbrida ou rollout gradual |
| Não trocar | B não supera A com significância ou piora em categorias críticas | Manter A; investigar B |

A recomendação final deve emergir dos **dados**, não desta tabela.

---

## Conexão com outras partes

| Parte | Conexão |
|---|---|
| Parte 1 (EDA) | Contexto dos dados e subgrupos |
| Parte 2 (Auditoria A) | Baseline de falhas do A — B deve ser comparado nesses pontos |
