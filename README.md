## Detecção de Fraudes em Transações Financeiras

Pesquisa necessária para a obtenção de Bacharel em Ciência da Computação pelo Centro Universitário Carlos Drummond de Andrade.

Autores:

Jairo Igor da Silva
Jorge Luis Quispe Flores
Kayham Cristoffer Guilhermino de Oliveira

Orientador: Prof. Ms. Marcus Vinicius Camillo Gália
Instituição: Centro Universitário Carlos Drummond de Andrade

## Objetivo

Avaliar modelos de classificação (Regressão Logística, Naive Bayes, Random Forest e
XGBoost) na detecção de fraudes em dados desbalanceados, comparando técnicas de
balanceamento (SMOTE vs. ponderação de classes) sob **split temporal**, para evitar
vazamento de dados futuros. Além da qualidade de detecção, o trabalho mede **tempo de
treino, latência por transação e throughput**, métricas pouco reportadas na literatura.

## Dataset - Credit Card Fraud Detection (Kaggle)

O arquivo não está no repositório devido ao tamanho. Baixe em
<https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud> e coloque o `creditcard.csv`
em `data/`.

## Metodologia

- **Split temporal** 70/15/15 (treino/validação/teste), ordenado por `Time`
- **Remoção de duplicatas** antes do split (1.081 registros)
- **Features**: V1–V28 e `Amount`. `Time` é usado só para ordenar o split e **não** entra
  como feature (valores inéditos no teste). Análise de sensibilidade com `sin/cos` da
  hora do dia
- **Pré-processamento**: `RobustScaler` apenas em `Amount`; SMOTE
  (`sampling_strategy=0.1`) e ponderação de classes aplicados **somente no treino**,
  dentro de pipelines (sem vazamento)
- **Limiar de decisão** escolhido na validação (máximo F2) e congelado para o teste
- **Métricas**: PR-AUC, F2, Precision, Recall e MCC, com IC de 95% por bootstrap (500
  reamostragens) no teste
- **Validação cruzada temporal**: janela expansiva (5 folds) sobre treino + validação,
  com PR-AUC
- **Desempenho computacional**: tempo de treino; latência por transação (modelo isolado e
  pipeline completo, 1 thread); throughput em lote

## Estrutura do repositório

```
data/         creditcard.csv (não versionado)
notebooks/    note.ipynb (EDA, avaliação, sensibilidade, CV e benchmark)
results/      eda.png e CSVs com os resultados
```

## Como reproduzir (Windows)

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Baixe o dataset (ver acima) e execute as células de `notebooks/note.ipynb` em ordem.
`RANDOM_STATE = 42`. Em um notebook em ambiente comum, a execução completa leva por volta de quinze minutos (a CV e o RF com SMOTE são as partes mais lentas).

## Resultados principais (teste temporal)

| Modelo | PR-AUC teste | PR-AUC CV (média ± dp) | Treino | Latência do modelo (mediana) |
|---|---|---|---|---|
| LR + peso | 0,712 | 0,750 ± 0,076 | 1,8 s | 0,19 ms |
| NB | 0,056 | 0,072 ± 0,034 | 0,3 s | 0,33 ms |
| RF + peso | 0,775 | 0,781 ± 0,071 | 22 s | 11,07 ms |
| XGBoost + peso | 0,761 | 0,791 ± 0,053 | 5,8 s | 1,27 ms |

RF e XGBoost têm desempenho de detecção equivalente (ICs sobrepostos); o XGBoost tem
treino e latência por transação bem menores. SMOTE não trouxe ganho consistente sobre a
ponderação de classes. Detalhes e ICs em `results/`.

## Limitações

- Apenas 52 fraudes no teste e 55 na validação: métricas ruidosas e limiar instável
- Validação e teste cobrem apenas o fim do segundo dia (sem madrugada)
- Um único dataset (2 dias de transações europeias, features anonimizadas por PCA)
- Vazão variou de 2 a 4 vezes entre execuções na mesma máquina; interpretar como ordem
  de grandeza

## Ambiente de execução

Windows 11, CPU Intel com 8 núcleos lógicos, Python 3.12.6, numpy 2.5.4, pandas 3.0.6,
scikit-learn 1.9.1, xgboost 3.4.1, imbalanced-learn 0.14.2 (ver `results/ambiente.csv`).

