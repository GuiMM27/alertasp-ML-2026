# ALERTASP — RELATÓRIO DE IA & MACHINE LEARNING

**FIAP — Global Solution 2026.2 | CitizenTech**  
**Equipe:** Enzo Almeida (RM556900); Guilherme Moreira (RM557290); Gabriel De Mello (RM554421).

## 1. Problema, pergunta e fontes

O ALERTASP estuda alagamentos urbanos na capital paulista. A pergunta de ML é: **a chuva medida na estação INMET A701 (São Paulo–Mirante) até o início de um dia permite estimar a probabilidade de haver ao menos um *registro administrativo* de alagamento ou inundação em algum ponto do município nas 24 horas seguintes?** O alvo não descreve uma rua nem a ocorrência física não notificada.

Fontes públicas reais: **GeoSampa/Defesa Civil/SIGRC**, 14 GeoPackages de 2013–2026 (2026 parcial; atualização mensal, ETL sem sincronismo), disponíveis nas fichas de [Alagamento](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869) e [Inundação](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/9d08d20d-f6c7-463d-a349-c28c47babff2); e **INMET**, CSVs horários da estação A701 de 2013–2026, com 2026 até 31/10 ([portal oficial](https://portal.inmet.gov.br/dadoshistoricos)). Unidades, licença/condições, hashes e limitações constam em `dados/README.md`.

## 2. Preparação e método

Unidade de análise: **município × dia civil**, com alvo binário `houve_registro` (1 = ao menos um registro; 0 = não consta registro). Precipitação horária (mm) foi convertida de UTC para `America/Sao_Paulo`, respeitando horário de verão histórico. Foram tratadas sentinelas de ausência e dias com cobertura meteorológica insuficiente, **sem substituir dado ausente por chuva zero**.

Features: chuva do dia anterior, acumulados observados nos três e sete dias anteriores, pico horário e horas chuvosas no dia anterior, frações de cobertura das janelas e mês codificado ciclicamente. O dia imediatamente anterior exige cobertura horária completa; janelas de três e sete dias requerem, respectivamente, 2/3 e 5/7 dias válidos. Tais somas podem subestimar a chuva total em janelas incompletas. **Nunca se utiliza a chuva do dia-alvo**, prevenindo vazamento temporal.

Modelo: **Random Forest** (permitido pela FIAP), comparado a baselines de frequência global e sazonal mensal. Desenvolvimento em 2013–2022 (**3.585 dias**); quatro folds cronológicos restritos ao desenvolvimento; **12 combinações de três hiperparâmetros** (`n_estimators`, `max_depth`, `min_samples_leaf`). Melhor configuração: **300 árvores, profundidade 4 e mínimo de 20 observações por folha**, com *Average Precision* média de **0,6141** na validação cruzada. Calibração sigmoidal em 2023 (268 dias), verificação em 2024 (354), teste retrospectivo em 2025 (327) e avaliação complementar parcial em 2026 (231).

## 3. Resultados e interpretação

| Métrica | RF calibrado — treino | RF calibrado — 2025 | Baseline mensal — 2025 |
|---|---:|---:|---:|
| Average Precision (maior melhor) | 0,6344 | **0,6233** | 0,5682 |
| Brier Score (menor melhor) | 0,1610 | 0,1395 | **0,1387** |
| F1-macro (limiar 0,50) | 0,7128 | 0,6705 | **0,7312** |
| Balanced accuracy | 0,7041 | 0,6645 | **0,7323** |

Em 2025, a matriz de confusão do RF calibrado foi **[[218, 33], [41, 35]]** (linhas reais: sem/com registro; colunas previstas: sem/com registro): houve **41 falsos negativos entre 76 dias com registro**. A frequência de dias com registro no teste foi 23,24%. Em 2024, o RF calibrado teve AP **0,6777** e Brier **0,1336**; em 2026 parcial, AP **0,6912**, Brier **0,1829** e F1-macro **0,7157** (períodos com distribuições diferentes, não comparáveis como evolução de um mesmo teste).

O RF ordenou melhor os dias por chance de registro do que a referência mensal em 2025 (**AP maior**), porém suas **probabilidades foram ligeiramente piores no Brier Score**, e o baseline mensal foi superior em F1-macro. O desempenho no treino é otimista; a calibração em 2023 não constitui teste independente. Há limitação de generalização e a curva de calibração apresenta desvios.

## 4. Conclusão para o cidadão e limitações

Observou-se associação entre **chuva anterior medida na A701** e frequência de registros no GeoSampa, mas o modelo **não está validado para comunicar porcentagens operacionais de risco**. Um único pluviômetro não representa a chuva de cada bairro; registros SIGRC sofrem subnotificação, mudanças de cobertura e possíveis atrasos; `0` significa *sem registro*, não *sem alagamento*. Não há horário preciso de cada ocorrência, radar, dados hidrológicos locais ou validação prospectiva em novas temporadas. O modelo não determina vias seguras, não produz probabilidade por rua nem substitui avisos do CGE e da Defesa Civil. A contribuição aos ODS 11 e 13 (ODS 3 complementar) é **informacional e potencial**, sem impacto medido. Os resultados apoiam pesquisa histórica, não decisões de segurança.

**Reprodução:** notebook `.ipynb` executado; `requirements.txt`; `dados/README.md`; evidências de 12 combinações, métricas e gráficos em `evidencias/chuva_a701/` e `figuras/chuva_a701/`.
