#  Desafio de Ciência de Dados – IMDB (Lighthouse)

##  Objetivo

O objetivo deste projeto é analisar um conjunto de dados do **IMDB** contendo informações sobre 999 filmes e construir um modelo preditivo para estimar a nota **IMDB Rating** de novos filmes. Além disso, são realizadas análises exploratórias e estatísticas para identificar padrões que explicam o desempenho de filmes em termos de avaliação e popularidade.

---

## Estrutura do Repositório

```
├── Lighthouse_Desafio_Cientista_de_Dados_IMDB.ipynb   # Notebook com análises, EDA e modelagem
├── model_imdb.pkl                                     # Modelo final CatBoost treinado
├── requirements.txt                                   # Lista de dependências
├── desafio_indicium_imdb.csv                          # Base de dados original
└── README.md                                          # Este arquivo
```

---

## Instalação

Clone este repositório e instale as dependências:

```bash
git clone https://github.com/Sugaharaa/Projetos-de-Modelos-Preditivos.git
cd "Projetos-de-Modelos-Preditivos/Desafio Ciencia de Dados Lighthouse"
pip install -r requirements.txt
```

---

## Etapas do Projeto

### 1. Análise Exploratória de Dados (EDA)

* **Estrutural:** identificação de colunas, tipos e valores ausentes.
* **Univariada:** distribuições de cada variável (IMDB Rating, Runtime, Gross, Meta\_score, etc.).
* **Bivariada:** relações entre IMDB Rating e outras variáveis (número de votos, faturamento, metascore, ano, etc.).
* **Correlação:** matriz de correlação para variáveis numéricas.

### 2. Tratamento de Dados

* Correção de inconsistências (ex.: valores inválidos em Released\_Year).
* Conversão de variáveis para numéricas (`Runtime`, `Gross`).
* Aplicação de **log-transform** em `No_of_Votes` e `Gross` para reduzir assimetria.
* Padronização de valores ausentes em colunas categóricas.

### 3. Modelagem

Foram testados os seguintes algoritmos:

* Regressão Linear
* Ridge
* Lasso
* CatBoost Regressor (modelo final escolhido)

O **CatBoost** apresentou desempenho superior:

* R² ≈ **0.54** (Cross-Validation média)
* RMSE ≈ **0.18**
* MAE ≈ **0.14**

Motivos da escolha:

* Melhor performance em relação aos modelos lineares.
* Lida de forma nativa com variáveis categóricas.
* Maior robustez frente a valores ausentes e multicolinearidade.

### 4. Predição do Filme Exemplo

Foi realizada a previsão da nota IMDB para o filme **"The Shawshank Redemption" (1994)**, conforme solicitado no desafio.

---

## Uso do Modelo Treinado

Após instalar as dependências, carregue o modelo `.pkl` para realizar previsões:

```python
import joblib
import pandas as pd
import numpy as np
from catboost import Pool

# Carregar modelo
model = joblib.load("model_imdb.pkl")

#Criar função que ira utilizar o modelo
def prever_nota_imdb(filme_dict, modelo):

    num_cols = ["Released_Year", "Runtime", "Meta_score", "No_of_Votes", "Gross"]
    cat_cols = ["Certificate", "Genre", "Director", "Star1", "Star2", "Star3", "Star4"]

    ordered_cols = num_cols + cat_cols

    df = pd.DataFrame([filme_dict])

    # Pré-processamento
    df["Released_Year"] = pd.to_numeric(df["Released_Year"], errors="coerce")
    df["Runtime"] = df["Runtime"].str.replace(" min", "", regex=False).astype(float)
    df["Gross"] = df["Gross"].astype(str).str.replace(",", "", regex=False).astype(float)
    df["No_of_Votes"] = np.log1p(df["No_of_Votes"])
    df["Gross"] = np.log1p(df["Gross"])

    # Categorias como string
    for c in cat_cols:
        if c not in df.columns:
            df[c] = ""
    df[cat_cols] = df[cat_cols].fillna("NA").astype(str)

    # Reordenar colunas
    df = df[ordered_cols]

    # Índices das colunas categóricas
    cat_idx = [ordered_cols.index(c) for c in cat_cols]

    pool = Pool(
        data=df,
        cat_features=cat_idx,
        feature_names=ordered_cols
    )

    pred = modelo.predict(pool)
    return float(pred[0])

# Exemplo de filme
filme = {
    'Series_Title': 'The Shawshank Redemption',
    'Released_Year': '1994',
    'Certificate': 'A',
    'Runtime': '142 min',
    'Genre': 'Drama',
    'Overview': 'Two imprisoned men bond over a number of years, finding solace and eventual redemption through acts of common decency.',
    'Meta_score': 80.0,
    'Director': 'Frank Darabont',
    'Star1': 'Tim Robbins',
    'Star2': 'Morgan Freeman',
    'Star3': 'Bob Gunton',
    'Star4': 'William Sadler',
    'No_of_Votes': 2343110,
    'Gross': 28341469.0
}

# Transformar em DataFrame
filme_df = pd.DataFrame([filme])

predicao = prever_nota_imdb(filme, model)
print(f"Predição de nota IMDB: {predicao:.2f}")
```

---

## Resultados Principais

* Filmes com **mais votos** apresentam maior estabilidade e melhores notas.
* **Faturamento (Gross)** está fortemente relacionado ao número de votos, mas **não** à qualidade percebida.
* **Crítica especializada (Meta\_score)** tem baixa correlação com o público.
* **Ano de lançamento** influencia a crítica, mas pouco a avaliação do público.

---

## Autor

Projeto desenvolvido por **Lucas Sugahara Lima**, como parte do processo seletivo de **Cientista de Dados – Lighthouse**.
