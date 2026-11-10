# ALERTASP — RELATÓRIO DE IA & MACHINE LEARNING
**FIAP — Global Solution 2026.2 | CitizenTech**  
**Equipe:** Enzo Almeida (RM556900); Guilherme Moreira (RM557290); Gabriel De Mello (RM554421).

## Problema e dados
Pergunta: com chuva medida na estação São Paulo–Mirante (INMET A701) **até o início do dia**, qual a probabilidade estimada de ao menos um **registro administrativo** de alagamento ou inundação no município nas 24 horas seguintes? O rótulo não confirma ausência física nos dias sem registro; há risco de subnotificação e atraso administrativo. Fontes: GeoSampa/Defesa Civil/SIGRC (2013–2026, 2026 parcial; atualização mensal via ETL sem sincronismo) e INMET A701 (medições horárias UTC dos anos efetivamente presentes, convertidas para horário local). Links e detalhes no `dados/README.md`.

## Preparação e método
Unidade: cidade × dia civil. Alvo: ao menos um registro SIGRC de alagamento ou inundação no dia (1/0). Features: chuva total do dia anterior (mm), acumulada em 3/7 dias anteriores (mm), pico horário e horas com chuva do dia anterior, cobertura das janelas de 3/7 dias, codificação cíclica do mês. Nunca se utiliza chuva do dia-alvo. O dia anterior deve ter todas as horas válidas; acumulados de 3/7 dias somam apenas dias completos observados com cobertura mínima de 2/3 e 5/7, respectivamente, e incluem frações de cobertura como features. Essas somas não são totais exatos quando há falhas; nenhum ausente é convertido em zero. A estação A701 não representa cada bairro.

Treino/seleção de hiperparâmetros: 2013–2022 (3585 dias), validação cruzada cronológica em 4 folds **só dentro do desenvolvimento**; calibração sigmoidal em 2023 (268 dias), verificação em 2024 (354), teste retrospectivo em 2025 (327) e avaliação parcial de 2026 (231). Random Forest permitido pelo PDF. Teste de **12 combinações** de três parâmetros: `n_estimators`, `max_depth`, `min_samples_leaf`. Melhor conjunto: `{'max_depth': 4, 'min_samples_leaf': 20, 'n_estimators': 300}`; AP média no CV: **0.6141**. Baselines de frequência histórica global e sazonal mensal treinados somente no período de desenvolvimento.

## Resultados
| Medida | RF calibrado — treino | RF calibrado — 2025 | Baseline mensal — 2025 |
|---|---:|---:|---:|
| Average Precision | 0.6344 | 0.6233 | 0.5682 |
| Brier (↓ melhor) | 0.1610 | 0.1395 | 0.1387 |
| F1-macro (limiar 0,50) | 0.7128 | 0.6705 | 0.7312 |
| Balanced accuracy | 0.7041 | 0.6645 | 0.7323 |

2025: proporção observada com registro **23.2%**. 2026: avaliação complementar feita (231 dias válidos); veja metricas_por_periodo.csv. Gráficos de PR, calibração e matriz de confusão em `figuras/chuva_a701/`. Os resultados de treino são otimistas; a comparação com 2025, separada temporalmente, é o teste retrospectivo.

## Interpretação para o cidadão e limites
O modelo não superou todos os baselines no critério pré-definido; não é defensável apresentar os percentuais como estimativas confiáveis para uso pelo cidadão. Trata-se de **registro SIGRC na cidade**, não probabilidade de alagamento de uma rua; fonte administrativa sujeita a subnotificação e diferenças de cobertura, chuva da A701 não representa cada bairro, ausência de horário da ocorrência, dados INMET sujeitos a falhas e ausência de radar/previsão meteorológica. Não orienta rotas nem substitui CGE/Defesa Civil. ODS 11 e 13 (e ODS 3 complementar) apenas como contribuição informacional potencial, não impacto demonstrado.

**Reprodutibilidade:**  `requirements.txt`, manifesto SHA-256, versões, CSV de GridSearch e notebook executado. Confirmar a divergência PDF p. 31 (2–4 páginas) versus p. 33 (2 páginas) no Teams.
