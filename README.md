# AAnálise de Dados e Previsão de Inadimplência — Databridge Squad 19

## Descrição

Projeto de análise de risco de crédito sobre dados sintéticos do portfólio Databridge Squad 19.
Combina uma pipeline de dados em camadas (Bronze → Silver → Gold), análise exploratória em notebooks
e um dashboard interativo em Streamlit com simulador de previsão de inadimplência em tempo real.

## Objetivos

- **Previsão de inadimplência** — 3 famílias de modelos (RF, GBM, XGBoost) com variantes global e segmentada
- **Análise de risco ambiental** — cruzamento de índices de seca/inundação com inadimplência por segmento
- **Detecção de fraude** — score de fraude e flags de alto risco
- **Performance do sistema OCR** — métricas de qualidade documental no onboarding
- **Segmentação de clientes** — perfil financeiro e comportamental por segmento

## Estrutura do Projeto

```
BB_data_analysis/
├── config.py                          # Constantes de Path (importar em vez de hardcode)
├── streamlit_app.py                   # Dashboard (10 páginas em 2 seções)
├── requirements.txt
├── data/
│   ├── raw/                           # Dados originais (não modificar)
│   ├── bronze/                        # Dados ingeridos
│   ├── silver/                        # Dados limpos e transformados
│   └── gold/
│       ├── gold_dashboard.csv         # Features completas para o dashboard
│       └── gold_model.csv             # Features para treino (sem colunas de leakage)
├── notebooks/
│   ├── bronze_ingestao.ipynb          # Raw → Bronze
│   ├── bronze_to_silver.ipynb         # Bronze → Silver
│   ├── eda/
│   │   ├── 03_eda.ipynb               # Análise exploratória geral
│   │   ├── 04_gold_Fraudes.ipynb      # Análise de fraude
│   │   ├── 05_gold_Inadimplencia.ipynb
│   │   ├── 06_gold_PerformanceSistema.ipynb
│   │   └── 07_gold_SegmentoDeClientes.ipynb
│   ├── silver_to_gold_dashboard.ipynb # Silver → gold_dashboard.csv
│   ├── silver_to_gold_model.ipynb     # Silver → gold_model.csv
│   ├── train_rf.ipynb                 # Treina RF Global + RF Segmentado
│   ├── train_gbm.ipynb                # Treina GBM Global + GBM Segmentado
│   ├── train_xgb.ipynb                # Treina XGB Global + XGB Segmentado
│   └── compare_models.ipynb           # Compara todos os 6 modelos → models/metrics.joblib
└── models/
    ├── rf/                            # Artefatos RF
    ├── gbm/                           # Artefatos GBM
    ├── xgb/                           # Artefatos XGBoost
    └── metrics.joblib                 # Métricas combinadas (gerado pelo compare_models)
```

## Instalação

```bash
git clone <url-do-repositorio>
cd BB_data_analysis
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Executar o Dashboard

```bash
source venv/bin/activate
streamlit run streamlit_app.py
```

Acesse em `http://localhost:8501`. O dashboard carrega `data/gold/gold_dashboard.csv` e os
artefatos em `models/` — garanta que os notebooks de treino foram executados antes.

## Pipeline de Dados

Execute os notebooks nesta ordem quando os dados brutos mudarem:

```
1. bronze_ingestao.ipynb
2. bronze_to_silver.ipynb
3. silver_to_gold_dashboard.ipynb   ← necessário para o dashboard
4. silver_to_gold_model.ipynb       ← necessário para retreinar modelos
```

## Treino dos Modelos

Execute em ordem após `silver_to_gold_model.ipynb`:

```
5. train_rf.ipynb
6. train_gbm.ipynb
7. train_xgb.ipynb
8. compare_models.ipynb             ← gera models/metrics.joblib
```

Cada notebook de treino exporta `global.joblib`, `metrics.joblib`, `seg_artifacts.joblib`
e 42 submodelos em `seg/` para o diretório do respectivo modelo.

## Resultados dos Modelos

Avaliação no conjunto de teste (hold-out estratificado de 20%, ~5.000 registros):

| Modelo | ROC-AUC | PR-AUC |
|---|---|---|
| GBM Global | 0,5885 | 0,2169 |
| RF Global | 0,5882 | 0,2207 |
| XGB Global | 0,5611 | 0,2046 |
| RF Segmentado | 0,5538 | 0,1922 |
| GBM Segmentado | 0,5302 | 0,1778 |
| XGB Segmentado | 0,5237 | 0,1726 |

**Dataset:** 24.974 contratos, 16,6% de inadimplência, 33 features.
O teto de desempenho (~0,59 ROC) é imposto pela ausência de preditores fortes
(histórico de pagamentos, score de bureau externo), não pelos algoritmos.

## Tecnologias

| Categoria | Bibliotecas |
|---|---|
| Pipeline de dados | pandas, numpy |
| Modelos | scikit-learn (RF, HistGradientBoosting), xgboost |
| Dashboard | Streamlit, Plotly, Matplotlib, Seaborn |
| Notebooks | Jupyter Lab |
| Persistência | joblib |

## Equipe — Squad 19

- Marcello Augusto (MGT-21)
- Paulo Nery (Paulo-nf)
- Luis Felipe (LuizMXavier)
- Pedro Sol
- Luiz Henrique
- Matheus Conolly
