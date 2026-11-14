# Detecção de Fraudes em Transações Financeiras

Pesquisa necessária para a obtenção de Bacharel em Ciência da Computação pelo Centro Universitário Carlos Drummond de Andrade.

**Autores:**

- Jairo Igor da Silva
- Jorge Luis Quispe Flores
- Kayham Cristoffer Guilhermino de Oliveira

**Orientador:** Prof. Ms. Marcus Vinicius Camillo Gália<br>
**Instituição:** Centro Universitário Carlos Drummond de Andrade

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
data/           creditcard.csv (não versionado)
notebooks/      deteccao_fraudes_split_temporal.ipynb (EDA, avaliação, sensibilidade, CV, benchmark e figuras)
results/        figuras (eda.png, fig_*.png) e CSVs com os resultados
requirements.txt
```

## Como reproduzir (Windows)

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

1. Baixe o dataset (ver acima) e coloque `creditcard.csv` em `data/`.
2. Abra `notebooks/deteccao_fraudes_split_temporal.ipynb` com o diretório de trabalho do
   kernel na pasta `notebooks/` (os caminhos são relativos: `../data` e `../results`).
3. O notebook exige a fonte **Times New Roman** instalada (padrão do TCC). Em sistemas
   sem ela, instale-a ou remova o `assert` da célula de configuração.
4. Execute as células em ordem. `RANDOM_STATE = 42`. A execução completa levou cerca de
   14 minutos na máquina descrita abaixo (a validação cruzada, o RF com SMOTE e o
   benchmark de inferência são as partes mais lentas).

## Resultados principais (conjunto de teste temporal)

Teste: 42.559 transações, 52 fraudes. RL = regressão logística; NB = Naive Bayes; RF =
floresta aleatória. IC de 95% por bootstrap (500 reamostragens); limiar escolhido na
validação (máximo F2). CV = validação cruzada temporal (5 folds), desvio-padrão amostral.

| Modelo | PR-AUC [IC 95%] | F2 | MCC | Precisão | Revocação | PR-AUC na CV | Treino |
|---|---|---|---|---|---|---|---|
| RL (peso) | 0,712 [0,560; 0,841] | 0,754 | 0,794 | 0,864 | 0,731 | 0,750 ± 0,085 | 2,1 s |
| RL (SMOTE) | 0,656 [0,513; 0,796] | 0,690 | 0,685 | 0,679 | 0,692 | 0,742 ± 0,074 | 1,8 s |
| NB | 0,056 [0,036; 0,078] | 0,233 | 0,201 | 0,067 | 0,615 | 0,072 ± 0,039 | 0,2 s |
| NB (SMOTE) | 0,055 [0,036; 0,077] | 0,237 | 0,203 | 0,068 | 0,615 | 0,076 ± 0,039 | 0,3 s |
| RF (peso) | 0,775 [0,663; 0,876] | 0,720 | 0,681 | 0,619 | 0,750 | 0,781 ± 0,079 | 20,9 s |
| RF (SMOTE) | 0,777 [0,660; 0,876] | 0,712 | 0,635 | 0,513 | 0,788 | 0,791 ± 0,072 | 38,4 s |
| XGBoost (peso) | 0,761 [0,641; 0,864] | 0,771 | 0,806 | 0,867 | 0,750 | 0,791 ± 0,059 | 5,0 s |
| XGBoost (SMOTE) | 0,770 [0,648; 0,872] | 0,760 | 0,813 | 0,905 | 0,731 | 0,790 ± 0,047 | 7,2 s |

### Custo de inferência

Mediana de 3 execuções completas. Latência medida por transação, com 1 thread (modelo
isolado); vazão em lote no conjunto de teste. O pipeline completo inclui cerca de 2 ms de
pré-processamento da pilha scikit-learn/pandas, custo da implementação e não dos algoritmos.

| Modelo | Latência do modelo: mediana / p95 / p99 (ms) | Pipeline completo, mediana (ms) | Vazão, 1 thread (tx/s) | Vazão, 8 núcleos (tx/s) |
|---|---|---|---|---|
| RL | 0,09 / 0,21 / 0,48 | 2,20 | 4,80 milhões | 3,95 milhões |
| NB | 0,14 / 0,37 / 0,71 | 2,44 | 1,12 milhão | 1,13 milhão |
| RF | 4,89 / 7,42 / 9,53 | 8,11 | 207 mil | 526 mil |
| XGBoost | 0,53 / 1,28 / 2,14 | 3,03 | 318 mil | 1,05 milhão |

### Sensibilidade ao ciclo diário (RF + SMOTE)

| Atributos | PR-AUC [IC 95%] | F2 | MCC |
|---|---|---|---|
| Sem `Time` | 0,777 [0,660; 0,876] | 0,712 | 0,635 |
| Com seno/cosseno da hora | 0,766 [0,645; 0,865] | 0,717 | 0,676 |

### Resumo

- RF, XGBoost e RL (peso) têm desempenho de detecção comparável (ICs sobrepostos); o NB é
  claramente inferior.
- SMOTE não trouxe ganho consistente sobre a ponderação de classes e dobrou o treino do RF.
- O ciclo diário não trouxe ganho detectável.
- O XGBoost treina cerca de 4 vezes mais rápido e tem latência de modelo cerca de 9 vezes
  menor que o RF.

Detalhes, ICs e figuras em `results/`.

## Limitações

- Apenas 52 fraudes no teste e 55 na validação: métricas ruidosas e limiar instável.
- Validação e teste cobrem apenas o fim do segundo dia (sem madrugada).
- Um único dataset (2 dias de transações europeias, atributos anonimizados por PCA).
- Hiperparâmetros fixos, sem otimização.
- Análise de sensibilidade feita apenas com RF + SMOTE.
- Sem testes estatísticos pareados entre modelos.
- Vazão: variou de 2 a 4 vezes em medições preliminares; nas 3 execuções finais, a
  variação foi de 4% a 8% (RL, RF e XGBoost) e de cerca de 35% (NB). Interpretar como
  ordem de grandeza.

## Ambiente de execução

Windows 11, CPU Intel com 8 núcleos lógicos, Python 3.12.6, numpy 2.5.4, pandas 3.0.6,
scikit-learn 1.9.1, xgboost 3.4.1, imbalanced-learn 0.14.2 (ver `results/ambiente.csv`).

## Versão

Versão utilizada no Trabalho de Conclusão de Curso: tag `v1.0`.