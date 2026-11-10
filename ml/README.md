# ALERTASP · Inteligência Artificial & Machine Learning

**Estimativa retrospectiva de registros municipais de alagamento e inundação a partir da precipitação observada.**

Este módulo investiga a pergunta:

> A chuva medida na estação INMET A701 até o início de um dia ajuda a estimar a possibilidade de haver **pelo menos um registro administrativo** de alagamento ou inundação em algum ponto do município de São Paulo nas próximas 24 horas?

A unidade de análise é **São Paulo (município) × dia civil**. O alvo `houve_registro` vale 1 se existe um registro no GeoSampa naquele dia e 0 se **não consta registro**. Isso não equivale à ocorrência física real de alagamento, nem permite estimar risco por bairro ou via.

## Dados e preparação

| Fonte | Cobertura utilizada | Papel no experimento |
|---|---|---|
| [GeoSampa / Defesa Civil / SIGRC](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869) | 2013–2026 (2026 parcial) | Datas e categorias `ALAGAMENTO` e `INUNDACAO` |
| [INMET — históricos](https://portal.inmet.gov.br/dadoshistoricos) | 2013–2026 (último CSV até 31/10/2026) | Precipitação horária da estação A701, São Paulo–Mirante |

Os horários UTC do INMET são convertidos para `America/Sao_Paulo`, incluindo as mudanças históricas de horário de verão. Entram no modelo somente chuva **anterior ao dia-alvo** (1, 3 e 7 dias), pico horário e horas chuvosas do dia anterior, cobertura das janelas e codificação cíclica do mês. Medições ausentes não são tratadas como chuva zero; a amostra é filtrada de acordo com a cobertura disponível.

O inventário, as unidades, as regras de limpeza, as condições de reutilização e o manifesto de integridade estão descritos em [dados/README.md](dados/README.md).

## Método

- **Modelo:** `RandomForestClassifier`, com 12 combinações de `n_estimators`, `max_depth` e `min_samples_leaf` em quatro folds cronológicos de desenvolvimento.
- **Referências:** baseline de frequência global e baseline sazonal mensal.
- **Calibração:** regressão logística sigmoidal em um ano separado, sem reajustar os parâmetros do Random Forest com o teste final.
- **Métricas:** Average Precision, Brier Score, ROC-AUC, precision, recall, F1, F1-macro, balanced accuracy, curvas de calibração e matrizes de confusão.

| Período | Papel | Dias elegíveis |
|---|---|---:|
| 2013–2022 | Desenvolvimento e validação temporal | 3.585 |
| 2023 | Calibração probabilística | 268 |
| 2024 | Verificação adicional | 354 |
| 2025 | Teste retrospectivo | 327 |
| 2026 (parcial) | Avaliação complementar | 231 |

A seleção de hiperparâmetros usa exclusivamente os dados de 2013–2022. Os anos posteriores são tratados separadamente; os resultados são retrospectivos, não uma validação operacional futura.

## Resultados

A melhor configuração foi `n_estimators=300`, `max_depth=4` e `min_samples_leaf=20`, com **Average Precision média de 0,6141** na validação cruzada.

| Métrica em 2025 | Random Forest calibrado | Baseline mensal |
|---|---:|---:|
| Average Precision (maior é melhor) | **0,6233** | 0,5682 |
| Brier Score (menor é melhor) | 0,1395 | **0,1387** |
| F1-macro (limiar 0,50) | 0,6705 | **0,7312** |
| Balanced accuracy | 0,6645 | **0,7323** |

Na matriz de confusão de 2025, o Random Forest calibrado identificou **35 dos 76 dias com registro**, deixando 41 falsos negativos (além de 33 falsos positivos). Em 2026 parcial, obteve AP 0,6912 e Brier 0,1829; essa comparação entre anos não demonstra evolução do modelo.

**Interpretação:** o Random Forest melhora a ordenação dos dias por possibilidade de registro (AP), mas não supera o baseline mensal na qualidade das probabilidades (Brier) nem no F1-macro no teste de 2025. Portanto, suas porcentagens são **saídas experimentais retrospectivas**, não alertas confiáveis para o público.

O estudo completo e suas limitações estão em [RELATORIO_ML.md](RELATORIO_ML.md). Os resultados detalhados estão em [evidencias/chuva_a701/](evidencias/chuva_a701/) e os gráficos em [figuras/chuva_a701/](figuras/chuva_a701/).

## Executar no Google Colab

1. Baixe ou clone este repositório. Copie seu conteúdo para `Meu Drive/ALERTASP_GS_2026/`, preservando a pasta `ml/`.
2. Confira os arquivos originais nos caminhos a seguir:

   ```text
   ml/dados/brutos/SIRGAS_GPKG_riscoocorrencia.zip
   ml/dados/brutos/inmet/A701_MIRANTE/*.CSV  (14 arquivos, 2013–2026)
   ```

3. Abra [01_ALERTASP_PREVISAO_CHUVA_A701.ipynb](01_ALERTASP_PREVISAO_CHUVA_A701.ipynb) no [Google Colab](https://colab.research.google.com/) e selecione **Ambiente de execução → Executar tudo**. Use CPU e autorize o acesso ao Drive.
4. O notebook lê as fontes, valida a cobertura, constrói o painel diário, executa o GridSearchCV, calibra as probabilidades e salva métricas, gráficos e evidências nas subpastas `chuva_a701/`.
5. Confira as saídas do notebook executado e o arquivo `requirements.txt`, que registra as dependências do ambiente utilizado.

O caminho padrão utilizado no Colab é `/content/drive/MyDrive/ALERTASP_GS_2026/ml`. Para executar localmente com Jupyter, configure a variável de ambiente `ALERTASP_BASE_DIR` para a pasta `ml/`. O repositório inclui um notebook com resultados salvos, mas a execução depende dos arquivos e das bibliotecas instaladas.

## Estrutura

```text
ml/
├── 01_ALERTASP_PREVISAO_CHUVA_A701.ipynb
├── README.md
├── RELATORIO_ML.md
├── requirements.txt
├── prompts.md
├── dados/
│   ├── README.md
│   ├── brutos/
│   └── processados/chuva_a701/
├── evidencias/chuva_a701/
└── figuras/chuva_a701/
```

## Limites e transparência

- Um único pluviômetro não representa todos os bairros da capital.
- `houve_registro = 0` significa que não consta registro oficial, e não que a rua estava seca.
- O GeoSampa disponibiliza dados históricos atualizados mensalmente; não é uma fonte de ocorrência em tempo real.
- As porcentagens não estão validadas para orientar deslocamentos, substituir alertas oficiais ou indicar risco por via.
- O artefato `.joblib`, se distribuído, é experimental; carregue apenas arquivos cuja origem seja confiável.

**Equipe:** Enzo Almeida (RM556900), Guilherme Moreira (RM557290) e Gabriel De Mello (RM554421).  
*FIAP · Global Solution 2026.2 · CitizenTech*
