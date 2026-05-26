# Análise de Atrasos de Voos - TC3 FIAP

## Visão Geral

Este projeto realiza uma análise completa de padrões de atrasos de voos nos Estados Unidos, desde a exploração de dados até a construção de modelos de regressão logística binária e gradient boosting.

## Vídeo
https://drive.google.com/file/d/1Sr3BC85OJBqASlYFiBbc9ABDCH72axQl/view?usp=sharing

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
- Companhias com maior chance taxa de atraso: NK, F9 e B6
- O outono tem menos chance de atraso
- Sexta-feira tem menos chance de atraso

**Métricas de avaliação:**
- ROC: 0.5366
- Coeficiente GINI: 7,32%
- Sensitividade: 10,91%
- Especificidade: 88,32%%
- Acurácia: 79,75%

---

### 3. tc3_xgboost.ipynb
**Modelo XGBoost (Gradient Boosting)**

Implementa um classificador XGBoost para comparação com regressão logística:
- 100 árvores com profundidade máxima 10
- Taxa de aprendizado: 0.01
- Tratamento de desbalanceamento de classes com `scale_pos_weight`
- Divisão treino/teste: 80/20 (4.57M treino, 1.14M teste)

**Performance:**
- ROC teste: 0.6301
- Coeficiente GINI teste: 26,02%
- Sensitividade treino: 69.35%
- Sensitividade teste: 68.57%
- Acurácia treino: 51.83%
- Acurácia teste: 51.79%

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
