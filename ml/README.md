# ALERTASP — IA & Machine Learning | Modelo de registro municipal condicionado à chuva

**Equipe:** Enzo Almeida (RM556900), Guilherme Moreira (RM557290), Gabriel De Mello (RM554421)  
**FIAP GS CitizenTech 2026.2 — Entrega de IA & ML**

## Pergunta que o notebook responde

> Dadas as observações de precipitação na estação INMET A701 **até 00h de um dia**, qual a estimativa de **ao menos um registro** de alagamento ou inundação da Defesa Civil **em algum ponto da cidade de São Paulo** nesse dia?

**Diferença essencial:** os dados não permitem concluir risco de uma rua, bairro, trajeto ou previsão da intensidade física dos alagamentos. `y=0` significa **sem registro publicado** nesse dia — não “não aconteceu”. A interface de alertas do ALERTASP é um projeto separado.

## Fontes oficiais / período

- **GeoSampa/Defesa Civil/SIGRC:** 14 GPKGs, 2013–2026 (2026 parcial), 12.625 registros das classes `ALAGAMENTO` e `INUNDACAO`. [Alagamento](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869) · [Inundação](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/9d08d20d-f6c7-463d-a349-c28c47babff2). Atualização mensal / ETL sem sincronismo; histórico **não** é tempo real. Prefeitura indica [CC BY-SA 4.0 para dados geoespaciais do GeoSampa](https://prefeitura.sp.gov.br/web/licenciamento/w/licen%C3%A7a-para-uso-de-dados-do-geosampa).
- **INMET:** 14 CSVs originais de precipitação horária na estação **A701 São Paulo–Mirante**, 2013–2026 (2026 até 31/10), extraídos do [portal](https://portal.inmet.gov.br/dadoshistoricos). Unidade de chuva: mm por hora; UTC convertido para horário de São Paulo, respeitando horário de verão histórico. Dados brutos com possíveis falhas; [orientação INMET](https://portal.inmet.gov.br/noticias/saiba-como-acessar-os-dados-meteorol%25C3%25B3gicos-dispon%25C3%25ADveis-no-site-do-inmet).

Detalhamento, dicionário, SHA-256 e limitações no [dados/README.md](dados/README.md) e [evidencias/chuva_a701/manifesto_sha256.csv](evidencias/chuva_a701/manifesto_sha256.csv).

## Arquivos da entrega

- `01_ALERTASP_PREVISAO_CHUVA_A701.ipynb`: notebook executado, incluindo tabelas, gráficos e conclusão final.
- `RELATORIO_ML.md`: relatório compacto com problema, método, avaliação e cidadania. `RELATORIO_ML_2_PAGINAS.pdf`: visualização complementar.
- `requirements.txt`: versões Python utilizadas na execução local. Notebook possui célula de instalação compatível com Colab.
- `dados/README.md`: inventário, condições de uso, unidades, política de ausências e dicionário.
- `evidencias/chuva_a701/`: GridSearchCV, métricas, partições, versões e hashes, artefato experimental `.joblib`.
- `figuras/chuva_a701/`: comparação treino/teste, matrizes, curvas de calibração e precisão-revocação.
- `prompts.md`: documentação da IA como copiloto e revisões, sem atribuir resultados inventados.

## Executar no Google Colab (reprodutível)

1. Faça upload deste repositório ou do pacote ZIP para `Meu Drive/ALERTASP_GS_2026/` preservando **a pasta `ml/`**.
2. Confira os 14 CSVs no caminho `ml/dados/brutos/inmet/A701_MIRANTE/`, e um ZIP `SIRGAS_GPKG_riscoocorrencia.zip` em `ml/dados/brutos/`.
3. Abra `ml/01_ALERTASP_PREVISAO_CHUVA_A701.ipynb` no [Google Colab](https://colab.research.google.com/) e execute **Ambiente de execução → Executar tudo**, com CPU. Autorize o Drive.
4. O notebook valida o código A701, as horas, os anos, a integridade dos dados, gera o painel e os folds de desenvolvimento. Executa 12 combinações × 4 folds, calibra em 2023 e avalia 2024/2025/2026.
5. Compare as métricas com as tabelas e CSVs distribuídos. O notebook salva novos CSVs e gráficos no Drive e regenera `RELATORIO_ML.md`; **se você editou o relatório manualmente, guarde backup antes de reexecutar**.
6. Preserve os outputs do `.ipynb` quando enviar ao GitHub, confira a visualização na página do arquivo e submeta o link junto ao vídeo não listado de até cinco minutos no Teams.

**Executar localmente:** instale `pip install -r ml/requirements.txt` e use `ALERTASP_BASE_DIR=/caminho/para/ALERTASP_GS_2026/ml` ao executar o notebook com Jupyter. Os artefatos são experimentais, não de produção.

## Metodologia sem vazamento

| Etapa | Período | Uso |
|---|---|---|
| Desenvolvimento | 2013–2022 | Treino + 4 folds temporais; Grade: 12 combinações de 3 hiperparâmetros |
| Calibração | 2023 | Sigmoide sobre escores, sem treinar novamente o RF |
| Verificação | 2024 | Diagnóstico sem reajustar hiperparâmetros |
| Teste retrospectivo | 2025 | Avaliação fora do desenvolvimento (já observada durante o projeto) |
| Complementar parcial | 01–09/2026 | Avaliação exploratória, registros com atualização mensal |

As features incluem apenas chuva do **dia anterior** e somas observadas de 3/7 dias anteriores, suas coberturas, pico horário anterior, horas com chuva e mês cíclico. Não se usa chuva do dia-alvo. Ausências nunca são convertidas em zero.

## Resultado — estudo executado

- **Random Forest escolhido:** `n_estimators=300`, `max_depth=4`, `min_samples_leaf=20`. Média de **Average Precision na CV = 0,6079**.
- **2025, 327 dias elegíveis:** RF calibrado AP **0.6221**, Brier **0.1418**, F1-macro **0.6681**. Baseline sazonal mensal AP **0.5682**, Brier **0.1387**, F1-macro **0.7312**.
- **2026 parcial, 231 dias:** RF calibrado AP **0.6947**, Brier **0.1827** e F1-macro **0.7019**.

A ordenação relativa de dias foi melhor que a do baseline mensal em 2025 (AP), mas **a qualidade probabilística do modelo calibrado não superou a referência sazonal em Brier**. Portanto, **não** apresentamos suas porcentagens como risco operacional nem risco de rua. Na investigação, usamos porcentagens apenas como exemplos retrospectivos com a devida interpretação.

## Observações de replicação

- Dia sem registro oficial **não prova** ausência de alagamento; o classificador aprende detecção administrativa.
- Falhas horárias do A701 podem excluir dias de análise; cobertura por ano e amostras está em `evidencias/chuva_a701/`.
- Uma estação é insuficiente para representar chuva espacial em toda a capital.
- Em 2026, usamos somente datas até 30/09, anteriores à revisão da camada em 06/10; a ausência de ocorrências depois disso **não comprova ausência real**.
- Os `requirements.txt` fixam versões da execução de referência; versões Colab podem precisar ajuste de ambiente, devendo isso ser documentado.
- **Não** executar artefatos `.joblib` de fontes não confiáveis. Eles foram gerados na execução deste estudo e servem apenas à reprodução da equipe.

## Rubrica e entrega

PDF FIAP p. 31–33: problema/dados reais 2, escolha modelo 1, preparação e divisão 2, >=10 combinações de 3 hiperparâmetros com CV apenas no treino 2, métricas treino/teste e gráficos/overfitting 2, leitura cidadã 1. **Divergência de extensão:** p.31: 2–4 páginas; p.33: 2 páginas; priorizamos 2 e recomendamos confirmar no Teams.

Não há garantia de nota máxima; a revisão final inclui notebook executado, links públicos válidos, vídeo de até 5 minutos no YouTube (não listado) e submissão até 19/11/2026 às 23h55.
