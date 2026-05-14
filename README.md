# Análise de Atrasos em Voos Domésticos nos EUA

## 📋 Descrição

Este projeto realiza uma análise exploratória e estatística de atrasos em voos domésticos nos Estados Unidos. Utiliza técnicas de clustering geográfico e análise de séries temporais para identificar padrões de atraso por companhia aérea e região.

## 🎯 Objetivos

- Identificar as principais causas de atraso em voos
- Analisar a distribuição de atrasos por companhia aérea
- Segmentar rotas em clusters geográficos
- Correlacionar regiões geográficas com tipos de atraso
- Fornecer insights para otimização operacional

## 📊 Estrutura do Projeto

### Dados de Entrada

- **flights.csv**: Dados de voos com informações sobre:
  - Horários programados e reais
  - Atrasos (partida, chegada, tempo de táxi)
  - Razões de atraso (sistema aéreo, segurança, companhia, aeronave, clima)
  - Informações da companhia aérea
  
- **airports.csv**: Dados geográficos dos aeroportos
  - Coordenadas (latitude/longitude)
  - Código IATA

## 🔍 Análise Realizada

### 1. Preparação de Dados
- Conversão de formato HHMM para duração
- Cálculo de timestamps para cada etapa do voo
- Categorização de dias da semana
- Limpeza de valores ausentes

### 2. Análise Descritiva
- **Proporção de voos atrasados**: 11.04% (atraso > 30 minutos)
- **Estatísticas principais**:
  - Média de atraso: 85 minutos
  - Mediana: 61 minutos
  - Desvio padrão: 73 minutos

### 3. Análise por Companhia Aérea
- Identificação de companhias com maior taxa de atraso
- NK: 19% de voos atrasados (maior taxa)
- Distribuição das causas de atraso por companhia

### 4. Clustering Geográfico
- **Método**: K-means em coordenadas de origem e destino
- **Clusters identificados**: 3 regiões principais
  - **Sudeste** (Southeast): 1.137 rotas, 1.384M voos
  - **Oeste** (West): 1.983 rotas, 1.950M voos
  - **Nordeste** (Northeast): 1.546 rotas, 1.888M voos

### 5. Correlação: Região × Causa de Atraso
- **Oeste**: Maior correlação com atrasos de companhia (0.504)
- **Nordeste**: Maior correlação com atrasos do sistema aéreo (-0.479)
- **Sudeste**: Sem correlação significativa

## 🛠️ Tecnologias Utilizadas

- **Polars**: Processamento eficiente de dados
- **Pandas**: Manipulação de dados tabulares
- **Scikit-learn**: Machine learning (K-means)
- **Matplotlib & Seaborn**: Visualizações

## 📈 Principais Descobertas

1. **Variação Regional**: Diferentes regiões têm padrões distintos de atraso
2. **Companhias Aéreas**: NK e F9 têm as maiores taxas de atraso
3. **Causas Predominantes**:
   - Late Aircraft (aeronave atrasada): principal causa
   - Airline Delay (atraso de companhia): segunda causa
   - Air System Delay (sistema aéreo): terceira causa

4. **Distribuição**: Atrasos concentram-se em faixa de 42-100 minutos (25º-75º percentil)

## 💻 Como Usar

### Pré-requisitos
```bash
pip install polars pandas scikit-learn matplotlib seaborn numpy
```

### Executar Análise
```bash
jupyter notebook tc3.ipynb
```

## 📁 Arquivos Principais

- `tc3.ipynb`: Notebook Jupyter com análise completa
- `flights.csv`: Dados de voos
- `airports.csv`: Dados de aeroportos
- `README.md`: Este arquivo

## 🔧 Estrutura da Análise Notebook

1. **Leitura e preparação de dados** (Células 1-5)
2. **Análise de distribuição de atrasos** (Células 6-10)
3. **Análise por companhia aérea** (Células 11-15)
4. **Clustering geográfico** (Células 16-20)
5. **Análise cruzada: Região × Causa de Atraso** (Células 21-25)

## 📝 Notas

- Os dados contêm ~0.27% de valores ausentes em ARRIVAL_DELAY, considerado negligenciável
- Outliers significativos observados em AA (American Airlines)
- Assimetria positiva (skewness: 4.68) indica cauda longa à direita

## 👤 Autor

Projeto desenvolvido para FIAP TC3

## 📄 Licença

MIT
