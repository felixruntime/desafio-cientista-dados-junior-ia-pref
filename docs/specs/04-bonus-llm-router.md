# Spec — Bônus: LLM + Uncertainty-Aware Router (MVP)

**Branch:** `feat/bonus-llm`  
**Notebook:** `notebooks/04_bonus_llm_router.ipynb`  
**Referência:** [README — Bônus](../../README.md#bônus-opcional---classificação-com-llm)

---

## Objetivo

Classificar **top-200 chamados difíceis** via API Rio Open (prompt text-only + CoT JSON), manter **Modelo B** no caminho AUTO (~96%), e comparar **router vs A vs B** com custo e limitações.

---

## Entregáveis MVP

- [ ] Seleção top-200 por score `u(i)` entre candidatos `D ∨ L`
- [ ] Prompt **text-only** (sem predições A/B — anti-anchoring)
- [ ] JSON com `raciocinio_logico`, `categoria`, `confianca`
- [ ] API síncrona (`httpx`), 1 retry, fallback → `pred_modelo_b`
- [ ] Cache CSV em `results/cache/llm_predictions.csv`
- [ ] Acc router vs B; Coverage; Acc@AUTO; Acc@LLM; taxa fallback
- [ ] 2 figuras em `results/figures/bonus/`
- [ ] Seção Bônus no README (~10 linhas)

---

## Router (offline)

| Sinal | Definição |
|---|---|
| D(i) | `pred_modelo_a != pred_modelo_b` |
| L(i) | concordam e `conf_modelo_b < 0.60` |
| u(i) | `(1-conf_B) + 0.3·D + 0.2·high_risk + 0.15·poda` |
| E | top_200({i : D∨L}, u) |

**AUTO:** pred_final = pred_modelo_b  
**LLM:** Rio API no texto; fallback B se API/JSON falhar

---

## Fora do escopo MVP

- asyncio, McNemar router vs B, LLM como juiz de A/B
- Fila REVIEW humana
- Portal rio-ai React

---

## Checklist de reprodução

```bash
cp .env.example .env   # RIO_API_KEY opcional se cache existir
pip install -r requirements.txt
jupyter notebook notebooks/04_bonus_llm_router.ipynb
```
