# Dados e dicionário — ALERTASP / INMET A701 + GeoSampa

## 1. Procedência e download

**GeoSampa/Defesa Civil/SIGRC:** dados oficiais `SIRGAS_GPKG_riscoocorrencia.zip` com 14 GeoPackages anuais 2013–2026, recorte de `ALAGAMENTO` e `INUNDACAO`. Fontes [Alagamento](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869) e [Inundação](https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/9d08d20d-f6c7-463d-a349-c28c47babff2). Referência de atualização dos metadados: **06/10/2026**, publicação mensal via ETL sem sincronismo. **Licença:** [Creative Commons BY-SA 4.0 dos dados do GeoSampa](https://prefeitura.sp.gov.br/web/licenciamento/w/licen%C3%A7a-para-uso-de-dados-do-geosampa); atribuição da Prefeitura obrigatória, compartilhamento nos mesmos termos quando cabível.

**INMET:** dados horários brutos estação automática **A701 / São Paulo–Mirante**, 14 CSVs de 2013 a 2026 (último até **31/10/2026**), [download oficial](https://portal.inmet.gov.br/dadoshistoricos), WMO A701, localização conforme arquivo: latitude -23,48333333°, longitude -46,61666666°. CSV com 8 linhas de metadados, separador `;`, codificação Latin-1 e número decimal com vírgula. **Unidade de precipitação: mm medidos em intervalo horário**; horas indicadas em UTC. `-9999` e células vazias indicam ausência, não chuva zero. [Nota oficial do INMET](https://portal.inmet.gov.br/noticias/saiba-como-acessar-os-dados-meteorol%25C3%25B3gicos-dispon%25C3%25ADveis-no-site-do-inmet) sobre falhas e dados não validados. **Condições de reutilização INMET:** a documentação pública consultada não fornece licença Creative Commons específica para esses CSVs; mantenha atribuição e confira os termos da fonte caso redistribua.

**Coleta:** arquivos oficiais obtidos e fornecidos pela equipe durante a GS. O [manifesto SHA-256](../evidencias/chuva_a701/manifesto_sha256.csv) permite verificar se o arquivo usado é o mesmo. Não acrescentamos registros sintéticos.

## 2. Cobertura e unidade de análise

- Observação bruta INMET: **uma estação × hora UTC**. Dados convertidos para dia civil **`America/Sao_Paulo`** com horário de verão histórico.
- Observação bruta GeoSampa: **uma ocorrência SIGRC com data/categoria**; eventos repetidos no mesmo dia contam como um único **dia com registro** para o alvo municipal.
- Observação de treinamento: **município inteiro × dia civil** (00h–23h59). O indicador `1` significa **ao menos um registro** de alagamento ou inundação no dia; `0` significa **não consta registro** na base, e não afirma ausência de alagamento físico.
- **Treinamento/calibração/teste:** anos completos até 2025, com filtros de qualidade da precipitação; **2026 apenas parcial até 30/09**, por revisão de 06/10 do GeoSampa. Último registro observado em 28/09; isso não garante completude dos dados administrativos.
- **Dados originais analisados:** 12.625 registros das duas categorias em 14 GPKGs; 1.382 dias distintos com registro de 2013 a 2025. Após filtragem meteorológica, 3.585 dias de desenvolvimento, 268 de calibração, 354 de verificação, 327 de teste retrospectivo e 231 de 2026 parcial.

## 3. Campos brutos de entrada

| Origem | Campo | Unidade | Descrição |
|---|---|---|---|
| GeoSampa | `data` | dia | Dia indicado no registro administrativo |
| GeoSampa | `ocorrencia` | categoria | `ALAGAMENTO` ou `INUNDACAO`; demais categorias ignoradas |
| INMET | `DATA ...` | data UTC | Campo com layout variando ao longo dos anos |
| INMET | `HORA (UTC)` / variante | hora UTC | Hora da medição, antes da conversão para fuso local |
| INMET | `PRECIPITAÇÃO TOTAL, HORÁRIO (mm)` | mm | Chuva observada na estação em intervalo horário |
| INMET | Latitude/Longitude do cabeçalho | graus | Coordenadas fixas da estação, não da ocorrência |

## 4. Campos calculados e preparação

| Feature | Unidade | Critério e disponibilidade |
|---|---|---|
| `houve_registro` | 0 ou 1 | Alvo na data local; ausência de registro não significa ausência física |
| `chuva_dia_mm` | mm | Chuva do dia corrente, **apenas para preparar os atributos futuros**; nunca entra no modelo do mesmo dia |
| `chuva_d1_mm` | mm | Soma do dia anterior **somente quando todas as horas locais esperadas foram medidas** |
| `chuva_d3_mm` | mm observados | Soma de dias completos entre os três dias anteriores; exige ao menos 2/3 válidos; pode subestimar total real |
| `chuva_d7_mm` | mm observados | Soma de dias completos entre os sete dias anteriores; exige ao menos 5/7 válidos; pode subestimar total real |
| `maior_chuva_hora_d1_mm` | mm/h (intervalo 1h) | Pico da hora no dia anterior com cobertura completa |
| `horas_chuvosas_d1` | h | Horas com chuva positiva no dia anterior, com cobertura completa |
| `cobertura_d3`, `cobertura_d7` | fração | Fração de dias completos disponíveis nas janelas anteriores; acompanha as somas com dados ausentes |
| `mes_sin`, `mes_cos` | -1 a 1 | Sazonalidade circular conhecida no começo do dia |
| `janela_valida` | bool | Todas as features possuem medidas válidas e critérios de cobertura atendidos |

**Sem vazamento:** chuva sempre deslocada um dia (ou mais), sem usar chuva do dia previsto, rótulos futuros ou dados do teste para selecionar hiperparâmetros. A seleção usa apenas os quatro folds 2013–2022; calibração só 2023; 2024 é diagnóstico, 2025 teste retrospectivo e 2026 avaliação parcial. O relatório não permite atribuir risco a um bairro.

## 5. Qualidade, viés e limitações

- Dias com chuva horária insuficiente são descartados da modelagem. Não imputamos NaN como 0; as janelas 3/7 dias usam somas observadas e frações de cobertura. Esses filtros alteram a composição das amostras (viés de seleção).
- A menor cobertura anual de features entre anos completos foi **73%** (ver arquivo [cobertura_meteorologica_por_ano.csv](../evidencias/chuva_a701/cobertura_meteorologica_por_ano.csv)); 2023 e 2025 têm ausências relevantes. A estação única também não cobre temporalmente e espacialmente toda a cidade.
- GeoSampa é atualização mensal/ETL sem sincronismo. Os registros podem sofrer subnotificação e diferenças de cobertura administrativa ao longo do tempo; não há referência completa de dias negativos.
- As proporções por faixas de precipitação são **frequências administrativas observadas**, não limites oficiais de alerta ou probabilidades físicas por rua.
- As análises são retrospectivas, com períodos específicos; generalização além da base depende de novos dados e avaliação.

## 6. Rastreabilidade e reprodutibilidade

Veja [README da disciplina](../README.md), `requirements.txt`, `evidencias/chuva_a701/versoes_execucao.json`, `evidencias/chuva_a701/manifesto_sha256.csv`, `evidencias/chuva_a701/gridsearch_12_combinacoes.csv`. Os nomes originais de todos os CSVs foram mantidos. A pasta `extras/massa_d_agua.geojson` foi fornecida no contexto da GS, mas **não faz parte do modelo A701 municipal**.
