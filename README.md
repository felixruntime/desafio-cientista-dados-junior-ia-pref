# Desafio Técnico — Cientista de Dados Júnior
## Time de IA — Casa Civil / IplanRio

**Autor:** [felixruntime](https://github.com/felixruntime)  
**Fork:** [github.com/felixruntime/desafio-cientista-dados-junior-ia-pref](https://github.com/felixruntime/desafio-cientista-dados-junior-ia-pref)

---

## Contexto

Este repositório contém a solução do desafio técnico de **Cientista de Dados Júnior** (Casa Civil / IplanRio): avaliar o classificador da **Central 1746** (modelo A em produção vs. modelo B candidato) sobre 5.000 chamados sintéticos e recomendar substituição com base em evidências.

> Os dados são totalmente sintéticos — nenhum dado real de cidadão foi utilizado.

---

## Abordagem

A análise foi organizada em **quatro notebooks**:

1. **EDA** (`01_analise_exploratoria.ipynb`) — padrões do corpus relevantes para classificação (desbalanceamento, texto bimodal, canal).
2. **Auditoria do modelo A** (`02_auditoria_modelo_a.ipynb`) — métricas com IC via bootstrap estratificado, modos de falha e calibração da confiança.
3. **Comparação A vs B** (`03_comparacao_e_recomendacao.ipynb`) — teste de McNemar (desenho pareado), trade-offs por categoria e recomendação para gestão.
4. **Bônus — LLM + router** (`04_bonus_llm_router.ipynb`) — router uncertainty-aware + Rio Open nos casos difíceis.

Figuras por etapa em `results/figures/{eda,auditoria,comparacao,bonus}/` (geradas ao executar os notebooks). Specs detalhadas em `docs/specs/`.

---

## Como reproduzir

```bash
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Execute os notebooks **nesta ordem** (kernel `.venv`), a partir da pasta `notebooks/` ou ajustando o `ROOT` do Setup:

1. `notebooks/01_analise_exploratoria.ipynb`
2. `notebooks/02_auditoria_modelo_a.ipynb`
3. `notebooks/03_comparacao_e_recomendacao.ipynb`
4. `notebooks/04_bonus_llm_router.ipynb` — roda **sem API** se `results/cache/llm_predictions.csv` existir; para novas predições LLM, copie `.env.example` → `.env` e defina `RIO_API_KEY`.

---

## Dados

Arquivo: **`dados/chamados_com_predicoes.csv`** (5.000 chamados, 11 colunas)

| Coluna | Descrição |
|---|---|
| `texto` | Texto do chamado |
| `categoria_real` | Rótulo humano (ground truth) |
| `pred_modelo_a` / `conf_modelo_a` | Predição e confiança do modelo A (produção) |
| `pred_modelo_b` / `conf_modelo_b` | Predição e confiança do modelo B (candidato) |
| `canal`, `bairro`, `data_abertura`, `id_chamado` | Metadados do chamado |

---

## Sumário executivo

**Principais achados**

- O corpus tem **8 categorias desbalanceadas** e textos **bimodais** (curtos ≤60 vs longos ≥150); o canal muda volume, não o mix de classes.
- O **modelo A** acerta ~**77%** dos chamados, mas falha de forma concentrada: confusão **esgoto → buraco** (~221 casos) e acurácia ~**59%** em textos curtos. A confiança declarada do A **não** discrimina acerto/erro.
- O **modelo B** chega a ~**87%** de acurácia (macro-F1 ~**0,85**). No teste **pareado de McNemar**, a vantagem do B é estatisticamente significativa (p ≪ 0,001). O B reduz drasticamente a confusão esgoto→buraco (**221 → 13**) e eleva a acurácia em textos curtos (~**85%**). Em **7 de 8** categorias o B melhora; a exceção é **poda de árvore** (acurácia cai de ~**78%** para ~**53%**). Diferente do A, a confiança do B é útil para triagem (erros concentram-se em baixa confiança).
- **Bônus (opcional):** router que mantém B em ~**96%** dos chamados (AUTO) e escala os **200 casos mais difíceis** para [Rio Open](https://github.com/prefeitura-rio/rio-ai); no subset escalado A ≈ **75%** vs B ≈ **0,5%** — fallback B alinhado ao AUTO, mas **subótimo** nesse corte (detalhes na seção Bônus).

**Recomendação**

Recomendamos **substituir o modelo A pelo modelo B** na Central 1746. Na amostra de 5.000 chamados, o B acerta cerca de **87%** dos casos frente a **77%** do A (diferença estatisticamente significativa no teste pareado). O B também corrige falhas graves do A — por exemplo, confunde bem menos esgoto com buraco e melhora muito o desempenho em textos curtos. **Risco:** na categoria de poda de árvore o B piora (acurácia cai de ~78% para ~53%). **Mitigação:** ao colocar o B em produção, monitorar semanalmente a poda de árvore e, se necessário, manter revisão humana ou regra híbrida só para essa categoria até o modelo ser ajustado. **Próximo passo:** rollout com painel das oito categorias e alerta se a poda cair abaixo do nível atual do A.

> Limitações: dados sintéticos — validar em produção antes do deploy definitivo.

---

## Resultados por etapa

| Etapa | Notebook | Principais entregas |
|---|---|---|
| EDA | `01_analise_exploratoria.ipynb` | Desbalanceamento, texto bimodal, canal vs mix de classes |
| Auditoria A | `02_auditoria_modelo_a.ipynb` | Métricas com IC (bootstrap), modos de falha, calibração |
| Comparação | `03_comparacao_e_recomendacao.ipynb` | McNemar pareado, trade-offs por categoria, recomendação |
| Bônus | `04_bonus_llm_router.ipynb` | Router uncertainty-aware + Rio Open (ver seção abaixo) |

---

## Bônus — LLM + Uncertainty-Aware Router

**Notebook:** `notebooks/04_bonus_llm_router.ipynb`  
**Spec:** `docs/specs/04-bonus-llm-router.md`  
**Referência Rio Open:** [prefeitura-rio/rio-ai](https://github.com/prefeitura-rio/rio-ai)

### Motivação

A Parte 3 recomenda **adotar o Modelo B**. O bônus não questiona essa conclusão: propõe um **desenho de implantação** — quando escalar um chamado difícil para um LLM, em vez de confiar apenas no B automático.

O B tem confiança útil para triagem, mas ainda erra em subgrupos (ex.: **poda de árvore**) e em casos em que **A e B discordam** (~1.702 chamados no pool `D∨L`). O router seleciona os **200 mais incertos** (top score `u`) para classificação via **Rio Open** (`rio-3.0-open-mini`); os demais **~96%** seguem em **AUTO** com `pred = B`.

### Como funciona o router

| Sinal | Definição |
|---|---|
| **D** | `pred_modelo_a ≠ pred_modelo_b` (divergência) |
| **L** | concordam e `conf_modelo_b < 0.60` (baixa confiança) |
| **u** | `(1 − conf_B) + 0.3·D + 0.2·high_risk + 0.15·poda` |
| **Escalonamento** | top-200 entre candidatos `D∨L`, ordenados por `u` |

- **AUTO (~4.800 chamados):** `pred_final = pred_modelo_b`
- **LLM (200 chamados):** API Rio no **texto do cidadão apenas** (anti-anchoring — A/B não entram no prompt)
- **Fallback (MVP):** falha de API, JSON inválido ou slug inválido → `pred_modelo_b` (alinhado ao AUTO; **subótimo** no subset escalado — ver Implicação de design)

### Prompt e integração

- System prompt com as 8 categorias e regras (esgoto ≠ buraco, poda vs fiação, etc.)
- Saída JSON: `raciocinio_logico` **antes** de `categoria` e `confianca` (chain-of-thought)
- Cliente síncrono (`httpx`), 1 retry, `temperature=0`
- Cache em `results/cache/llm_predictions.csv` para reproduzir **sem** `RIO_API_KEY`

### Reprodução do bônus

```bash
cp .env.example .env   # opcional: RIO_API_KEY para novas predições LLM
jupyter notebook notebooks/04_bonus_llm_router.ipynb
```

Com o cache commitado, o notebook roda end-to-end sem API. Para predições LLM reais, configure `.env` e **apague ou renomeie** o cache antes de reexecutar.

### Resultados (cache atual — sem API Rio)

| Métrica | Valor | Interpretação |
|---|---|---|
| Acc Modelo A / B / Router | ~77% / ~87% / ~87% | Router ≡ B enquanto LLM está em fallback |
| Coverage AUTO / LLM | 96% / 4% | 200 chamados escalados |
| Acc@AUTO (B) | ~90% | B performa bem onde não escala |
| Acc@LLM (A) | ~75% | No subset difícil (todos **D**), A ainda acerta mais |
| Acc@LLM (router) | ~0,5% | Fallback B no subset onde B já erra ~99,5% |
| Taxa fallback B | 100% | Cache gerado com `no_api_key` |
| Acc router (fallback A hipot.) | ~89,7% | Usar A nos 200 escalados recuperaria ~**+3 p.p.** globais |
| Custo estimado (com API) | ~116k tokens | 200 chamadas × (~500 in + ~80 out) |

Figuras: `results/figures/bonus/01_acc_router_vs_b.png`, `02_subset_escalado_breakdown.png`.

### Implicação de design

O score `u` prioriza `(1 − conf_B)` e divergência **D** — o top-200 concentra casos em que o **B já erra** (~99,5% no subset escalado). Por isso Acc@LLM(B) ≈ 0,5% reflete **seleção**, não só falha da API.

- **Fallback B:** coerente operacionalmente com AUTO e a recomendação global de adotar B, mas **pior escolha de acurácia** no caminho LLM quando o Rio Open está indisponível.
- **Refinamento baseado em evidência:** fallback **A** quando `D`, fila humana, ou LLM real — não implementado no MVP.
- Isso **não invalida** adotar B globalmente (87% vs 77%); expõe um gap de política no caminho escalado.

### Limitações

- Dados **sintéticos** — métricas absolutas não generalizam para produção.
- Top-200 são **só casos D** (divergência); casos L-only têm `u` menor e não entram no corte — o router captura **parcialmente** baixa confiança do B.
- Sem API Rio, o bônus demonstra **arquitetura e política de fallback**, não ganho real do LLM.
- Fallback para B no MVP é **subótimo** no subset escalado; refinamento documentado como trabalho futuro.

### Conexão com a recomendação principal

O router **não substitui** a troca A→B. Ele complementa: B no fluxo principal (~96%), LLM (ou fallback refinado) nos ~4% escalados por incerteza.

---

## Estrutura do repositório

```
desafio-cientista-dados-junior-ia-pref/
├── README.md
├── .env.example
├── notebooks/
│   ├── 01_analise_exploratoria.ipynb
│   ├── 02_auditoria_modelo_a.ipynb
│   ├── 03_comparacao_e_recomendacao.ipynb
│   └── 04_bonus_llm_router.ipynb
├── dados/
│   └── chamados_com_predicoes.csv
├── results/
│   ├── cache/
│   │   └── llm_predictions.csv
│   └── figures/
│       ├── eda/
│       ├── auditoria/
│       ├── comparacao/
│       └── bonus/
├── docs/
│   ├── SPECS.md
│   └── specs/
└── requirements.txt
```
