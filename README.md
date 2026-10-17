## Detecção de Fraudes em Transações Financeiras

Pesquisa necessária para a obtenção de Bacharel em Ciência da Computação pelo Centro Universitário Carlos Drummond de Andrade.

Autores:

Jairo Igor da Silva
Jorge Luis Quispe Flores
Kayham Cristoffer Guilhermino de Oliveira

Orientador: Prof. Ms. Marcus Vinicius Camillo Gália
Instituição: Centro Universitário Carlos Drummond de Andrade

## Objetivo

Avaliar modelos de classificação (Regressão Logística, Naive Bayes,
Random Forest e XGBoost) em dados desbalanceados, comparando técnicas de
balanceamento (SMOTE vs. ponderação de classes) e usando split temporal
para evitar vazamento de dados futuros.

## Dataset - Credit Card Fraud Detection (Kaggle)  

O arquivo não está no repositório devido do tamanho. Baixe em: <https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud> e coloque o `creditcard.csv` em `data/`.

## Metodologia

- Split temporal: treino / validação / teste, ordenados por `Time`
- Remoção de duplicatas antes do split
- Pré-processamento (`RobustScaler`) e balanceamento ajustados apenas no treino
- Limiar de decisão escolhido na validação
- Métricas: PR-AUC, F2, Precision, Recall e MCC

