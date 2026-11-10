# Dados utilizados — GeoSampa e INMET A701

Esta pasta reúne os dados públicos utilizados no experimento de Machine Learning do ALERTASP e documenta sua origem, cobertura, formato, tratamento e limitações. **Não foram utilizados dados sintéticos no treinamento ou na avaliação.**

## Fontes e condições de uso

### GeoSampa / Defesa Civil / SIGRC

- **Órgão:** Prefeitura do Município de São Paulo / Coordenadoria Municipal de Defesa Civil.
- **Conjunto:** série histórica de ocorrências de alagamento e inundação extraídas do SIGRC; pacote `SIRGAS_GPKG_riscoocorrencia.zip`, com 14 arquivos GeoPackage anuais (2013–2026).
- **Período:** 2013–2026, com 2026 parcial. Os metadados de referência foram revisados em 06/10/2026.
- **Atualização:** mensal, por ETL sem sincronismo; os registros não constituem monitoramento em tempo real.
- **Links oficiais:** [Alagamento](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869) e [Inundação](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/9d08d20d-f6c7-463d-a349-c28c47babff2).
- **Condições de uso:** [CC BY-SA 4.0 para dados geoespaciais GeoSampa](https://prefeitura.sp.gov.br/web/licenciamento/w/licen%C3%A7a-para-uso-de-dados-do-geosampa); manter atribuição e observar as condições de compartilhamento da fonte.

### INMET — estação A701 São Paulo–Mirante

- **Órgão:** Instituto Nacional de Meteorologia (INMET).
- **Conjunto:** 14 arquivos CSV anuais com medições de estação automática, de 2013 a 2026 (o CSV de 2026 cobre até 31/10/2026).
- **Referência espacial:** estação São Paulo–Mirante (código A701). Os metadados de localização variam ligeiramente ao longo da série e são auditados no notebook.
- **Variável utilizada:** precipitação total em intervalo horário, expressa em **milímetros (mm)**; não é uma leitura de chuva específica de cada bairro.
- **Horário e formato:** medições em UTC; arquivos com cabeçalhos de metadados, separador `;` e decimal com vírgula. Datas e nomes de colunas variam entre anos.
- **Download:** [INMET — dados históricos](https://portal.inmet.gov.br/dadoshistoricos).
- **Condições de uso:** manter a identificação do INMET e conferir as condições publicadas pelo órgão antes de redistribuições adicionais; não atribuímos aos CSVs uma licença específica não identificada.

Os arquivos originais devem permanecer sem edição em `brutos/`. O [manifesto de hashes SHA-256](../evidencias/chuva_a701/manifesto_sha256.csv) registra os arquivos efetivamente utilizados.

## Organização dos arquivos

```text
dados/
├── README.md
├── brutos/
│   ├── SIRGAS_GPKG_riscoocorrencia.zip
│   └── inmet/A701_MIRANTE/
│       └── 14 CSVs originais (2013–2026)
└── processados/chuva_a701/
    └── painel_diario_a701_auditoria.csv
```

## Unidade de análise e variável-alvo

Uma observação corresponde a **um dia civil no município de São Paulo**. A variável `houve_registro` é igual a 1 quando há ao menos um registro das categorias `ALAGAMENTO` ou `INUNDACAO` na base SIGRC naquele dia; é igual a 0 quando **não consta** registro dessas categorias. Vários registros no mesmo dia são consolidados no indicador diário municipal.

Essa definição não permite afirmar que dias com valor 0 foram dias sem alagamentos físicos: pode haver subnotificação, atraso ou diferenças de cobertura da administração pública.

## Dicionário dos principais campos

| Campo | Origem | Unidade | Significado |
|---|---|---|---|
| `data` | GeoSampa | Data | Dia associado ao registro administrativo |
| `ocorrencia` | GeoSampa | Categoria | `ALAGAMENTO` ou `INUNDACAO`; outras categorias não compõem o alvo |
| `HORA (UTC)` e variantes | INMET | Hora UTC | Horário de observação, convertido para `America/Sao_Paulo` |
| `PRECIPITAÇÃO TOTAL, HORÁRIO (mm)` e variantes | INMET | mm/intervalo de 1 h | Chuva registrada pela estação A701 |
| `houve_registro` | Derivado | 0/1 | Pelo menos um registro em São Paulo no dia |
| `chuva_d1_mm` | Derivado | mm | Chuva **do dia anterior**, exigindo cobertura horária completa |
| `chuva_d3_mm`, `chuva_d7_mm` | Derivado | mm observados | Acumulados dos 3/7 dias anteriores; podem ser parciais quando há falhas |
| `maior_chuva_hora_d1_mm` | Derivado | mm em 1 h | Maior medição horária do dia anterior |
| `horas_chuvosas_d1` | Derivado | h | Quantidade de horas com chuva no dia anterior |
| `cobertura_d3`, `cobertura_d7` | Derivado | Fração | Proporção de dias completos disponíveis nas janelas anteriores |
| `mes_sin`, `mes_cos` | Derivado | Adimensional | Sazonalidade conhecida no início do dia |

## Critérios de limpeza e cobertura

- Datas e horários são convertidos de UTC para o dia civil `America/Sao_Paulo`, considerando as transições históricas do horário de verão.
- Valores `-9999`, vazios e leituras inválidas representam ausência de dados; **não são convertidos em zero mm**.
- Para `chuva_d1_mm`, somente são usados dias anteriores com todas as horas locais esperadas e válidas.
- As janelas de 3 e 7 dias exigem pelo menos **2 de 3** e **5 de 7** dias válidos, respectivamente. As somas representam **chuva observada**, não uma extrapolação para os dias faltantes; a cobertura é incluída como variável separada.
- Chuva observada no dia que se pretende estimar **não entra nas features**. Apenas medições anteriores são utilizadas.
- Dias com features indisponíveis são removidos da modelagem. Esse filtro pode alterar a representatividade temporal dos dados.

## Cobertura da modelagem

| Conjunto | Período | Dias elegíveis |
|---|---|---:|
| Desenvolvimento / CV | 2013–2022 | 3.585 |
| Calibração | 2023 | 268 |
| Verificação | 2024 | 354 |
| Teste retrospectivo | 2025 | 327 |
| Complementar, parcial | 2026 | 231 |

A avaliação complementar de 2026 limita-se a datas com cobertura e registros considerados elegíveis no experimento; a série de ocorrências administrativa não é necessariamente completa após a data da atualização de referência. A cobertura meteorológica por ano está em [cobertura_meteorologica_por_ano.csv](../evidencias/chuva_a701/cobertura_meteorologica_por_ano.csv).

## Limitações

O INMET A701 é uma estação pontual, insuficiente para representar a precipitação em toda a cidade. Os registros SIGRC podem apresentar subnotificação e viés de cobertura. A associação entre chuva do dia anterior e frequência de registros é retrospectiva e **não define limites oficiais, risco por rua ou probabilidades operacionais confiáveis**.

Consulte a [metodologia completa](../README.md), o [relatório](../RELATORIO_ML.md) e os [resultados auditáveis](../evidencias/chuva_a701/).
