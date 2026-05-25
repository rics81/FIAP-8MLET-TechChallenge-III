# Análise de Atrasos de Voos - TC3 FIAP

## Visão Geral

Este projeto realiza uma análise completa de padrões de atrasos de voos nos Estados Unidos, desde a exploração de dados até a construção de modelos de regressão logística binária e gradient boosting.

## Estrutura do Projeto

```
tc3/
├── README.md                          # Este arquivo
├── tc3_data_explorer.ipynb           # Exploração exploratória dos dados
├── tc3_lbr.ipynb                     # Regressão logística binária
└── tc3_xgboost.ipynb                 # Modelo XGBoost
```

## Notebooks

### 1. tc3_data_explorer.ipynb
**Análise Exploratória dos Dados (EDA)**

Realiza a limpeza, transformação e exploração inicial dos dados:
- Leitura do dataset de voos (5.8M registros)
- Remoção de voos cancelados (89,884 voos)
- Remoção de registros com atraso ausente (15,187 voos)
- Análise descritiva dos atrasos de chegada
- Distribuição de atrasos por companhia aérea
- Clusterização de rotas usando K-Means (3 clusters: Nordeste, Sudeste, Oeste)
- Análise de correlação entre regiões geográficas e causas de atraso

**Features criadas:**
- `Day_of_week`: Dia da semana (Sun, Mon, Tue, Wed, Thu, Fri, Sat)
- `Season`: Estação do ano (win, spr, sum, aut)
- `Scheduled_departure_sin/cos`: Transformação trigonométrica da hora de partida
- `Scheduled_arrival_sin/cos`: Transformação trigonométrica da hora de chegada
- `Estimated_flight_time`: Tempo estimado de voo (minutos)
- `Distance`: Distância da rota (milhas)
- `Route_Cluster`: Cluster da rota (0, 1, 2)

**Saída:** `df_final_ml_features.parquet` (5.7M registros, 11 colunas)

---

### 2. tc3_lbr.ipynb
**Regressão Logística Binária (LBR)**

Implementa um modelo de regressão logística binária para prever atrasos:
- Variável alvo binária: `Delayed` (0: ≤30min, 1: >30min)
- Modelo: GLM com família Binomial (statsmodels)
- 11% dos voos têm atraso > 30 minutos (632,697 voos)

**Resultados principais:**
- Companhias com maior taxa de atraso: NK (19%), F9 (17%), B6 (16%)
- Estação de outono tem menos atrasos (-55.78% vs referência)
- Sexta-feira tem menos atrasos (-27.20% vs referência)
- Companhia WN tem 52.91% mais risco de atraso vs referência

**Métricas de avaliação:**
- ROC AUC: 0.7500 (excelente discriminação)
- Coeficiente GINI: 0.5000
- Curva ROC plotada com visualização clara

---

### 3. tc3_xgboost.ipynb
**Modelo XGBoost (Gradient Boosting)**

Implementa um classificador XGBoost para comparação com regressão logística:
- 100 árvores com profundidade máxima 10
- Taxa de aprendizado: 0.01
- Tratamento de desbalanceamento de classes com `scale_pos_weight`
- Divisão treino/teste: 80/20 (4.57M treino, 1.14M teste)

**Performance:**
- Acurácia treino: 51.62%
- Acurácia teste: 51.60%
- Recall teste: 68.81%
- AUC teste: 0.6299

**Features mais importantes:**
- Identificadas automaticamente pelo modelo
- Visualização em gráfico de barras horizontal

---

## Definições de Variáveis

### Variáveis Categóricas

| Variável | Valores | Referência (em modelos) |
|----------|---------|------------------------|
| `Day_of_week` | Sun, Mon, Tue, Wed, Thu, Fri, Sat | Sun |
| `Season` | win (inverno), spr (primavera), sum (verão), aut (outono) | win |
| `Airline` | 14 companhias aéreas (AS, AA, US, DL, NK, UA, HA, B6, OO, EV, F9, WN, MQ, VX) | WN |
| `Route_Cluster` | 0, 1, 2 (regiões geográficas) | 0 |

### Variáveis Quantitativas

| Variável | Descrição | Unidade | Intervalo |
|----------|-----------|---------|-----------|
| `Scheduled_departure_sin` | Seno da hora de partida | [-1, 1] | Transformação cíclica |
| `Scheduled_departure_cos` | Cosseno da hora de partida | [-1, 1] | Transformação cíclica |
| `Scheduled_arrival_sin` | Seno da hora de chegada | [-1, 1] | Transformação cíclica |
| `Scheduled_arrival_cos` | Cosseno da hora de chegada | [-1, 1] | Transformação cíclica |
| `Estimated_flight_time` | Tempo estimado de voo | minutos | 18-718 min |
| `Distance` | Distância da rota | milhas | 31-4,983 milhas |
| `Arrival_delay` | Atraso na chegada | minutos | -87 a 1,971 min |
| `Delayed` (alvo) | Indicador de atraso > 30min | binário | 0 ou 1 |

---

## Processamento de Dados

### Pipeline de Transformação

1. **Limpeza Inicial**
   - Remoção de voos cancelados (1.56% dos dados)
   - Remoção de registros com `ARRIVAL_DELAY` nulo (0.27% dos dados)

2. **Conversão de Data/Hora**
   - Combinação de YEAR, MONTH, DAY em timestamp
   - Conversão de HHMM (inteiro) para duração
   - Cálculo de tempos de chegada estimados e reais

3. **Feature Engineering**
   - **Transformação cíclica**: Horas do dia convertidas em seno/cosseno
     - Mantém a natureza cíclica de 24 horas
     - Evita descontinuidade artificial entre 23:59 e 00:00
   - **Tempo de voo**: Diferença entre chegada e partida estimadas
   - **Clustering de rotas**: K-Means em coordenadas de origem/destino

4. **Categorização de Atrasos**
   - **Modelo Binário**: Delayed = 1 se arrival_delay > 30 min
   - **Modelo Multinomial**: 
     - 0: not_delayed (< 0 min)
     - 1: lo_delay (0-15 min)
     - 2: mid_delay (15-30 min)
     - 3: high_delay (> 30 min)

---

## Principais Insights

### Distribuição de Atrasos
- **11.07%** dos voos têm atraso > 30 minutos
- Mediana de atraso (voos atrasados): 61 minutos
- Assimetria positiva (skewness = 4.68): cauda longa à direita
- 97.72% dos voos atrasados ficam dentro de 231 minutos

### Padrões por Companhia
| Companhia | Taxa de Atraso | Principal Razão |
|-----------|-----------------|-----------------|
| NK | 19% | Razões do sistema aéreo (57.27%) |
| F9 | 17% | Razões do sistema aéreo |
| B6 | 16% | Aeronave atrasada (59.47%) |
| WN | 14% | Aeronave atrasada (56.27%) |
| HA | 9% | Razões do sistema aéreo |

### Padrões por Dia da Semana
- **Quinta-feira**: -10.92% vs referência (menos atrasos)
- **Sexta-feira**: -27.20% vs referência (significativamente menos atrasos)
- **Domingo**: +6.25% vs referência (mais atrasos)

### Padrões por Estação
- **Outono**: -55.78% vs referência (significativamente menos atrasos)
- **Primavera**: -22.00% vs referência (menos atrasos)
- **Verão**: +1.02% vs referência (mais atrasos)

### Regiões de Rotas
- **Cluster 0 (Nordeste)**: 1,981 rotas, 1.97M voos, 6.67min atraso médio
- **Cluster 1 (Oeste)**: 1,563 rotas, 1.92M voos, 5.21min atraso médio
- **Cluster 2 (Sudeste)**: 1,090 rotas, 1.34M voos, 3.41min atraso médio

---

## Como Usar

### Requisitos
```bash
pip install polars pandas numpy scikit-learn statsmodels xgboost matplotlib seaborn
```

### Execução
1. Coloque `flights.csv` e `airports.csv` no diretório raiz
2. Execute os notebooks em ordem:
   - `tc3_data_explorer.ipynb` (gera `df_final_ml_features.parquet`)
   - `tc3_lbr.ipynb` (regressão logística binária)
   - `tc3_xgboost.ipynb` (modelo XGBoost)

### Interpretação de Resultados

**Regressão Logística Binária:**
- Coeficientes representam mudanças no log-odds de atraso
- Valores positivos aumentam probabilidade de atraso
- ROC AUC de 0.75 indica boa capacidade preditiva

**XGBoost:**
- Acurácia moderada (51.6%) reflete desbalanceamento de classes
- Recall alto (68.8%) indica capacidade de identificar voos atrasados
- Feature importance mostra quais variáveis mais contribuem

---

## Referências

- Documentação [Polars](https://www.pola.rs/)
- Documentação [XGBoost](https://xgboost.readthedocs.io/)
- Documentação [Statsmodels](https://www.statsmodels.org/)
- Documentação [Scikit-learn](https://scikit-learn.org/)

---

## Autor

Projeto de análise preditiva desenvolvido para FIAP TC3.

**Data:** Maio de 2026  
**Dataset:** US Flight Delays and Cancellations (2015)
