# Uso de IA no desenvolvimento do ALERTASP

A equipe utilizou assistentes de IA como apoio para planejamento técnico, exploração dos dados, revisão de código e interpretação dos resultados. Os arquivos oficiais vieram do GeoSampa e do INMET, e as métricas apresentadas foram obtidas com a execução do notebook sobre dados reais.

## Exemplos de solicitações efetivamente realizadas

> “Atue como cientista de dados/engenheiro de ML e orientador acadêmico. Ensine de forma didática e rigorosa, pois o objetivo é cumprir integralmente a rubrica da FIAP.”

**Finalidade:** organizar a pergunta de pesquisa, a leitura dos dados, a separação treino/teste e a validação cruzada conforme a disciplina.

> “Com base na chuva observada até o início de um dia e nas características históricas de determinada região de São Paulo, qual é a probabilidade estimada de haver pelo menos um registro de alagamento ou inundação nessa região durante as próximas 24 horas?”

**Finalidade:** reformular a pergunta inicial de classificação histórica para estimativa de registro futuro; o escopo foi posteriormente delimitado ao município inteiro.

> “Vamos fazer só da capital [...] ”

**Finalidade:** delimitar as fontes ao INMET A701 e às ocorrências GeoSampa de São Paulo, e estruturar um notebook reproduzível.

> “Analise tudo que foi proposto a nossa ideia, tudo o que já construímos [...] vamos pelo 10.”

**Finalidade:** realizar revisão da documentação e checagem de requisitos acadêmicos.

## Decisões técnicas revisadas

- **Sem dados sintéticos:** somente observações meteorológicas e registros públicos reais no experimento.
- **Disponibilidade temporal:** chuva usada como entrada sempre é anterior ao dia-alvo; UTC é convertido para o horário local.
- **Qualidade:** ausência meteorológica não é tratada como precipitação zero; os critérios de cobertura ficam documentados.
- **Validação:** desenvolvimento (2013–2022), calibração separada (2023), verificação (2024) e avaliação retrospectiva (2025), além de 2026 parcial.
- **Interpretação:** o Random Forest teve AP maior que o baseline sazonal em 2025, mas Brier Score pior; suas porcentagens não são apresentadas como alertas operacionais.

O uso de IA não substitui a conferência das fontes, a revisão do código nem a responsabilidade da equipe pela explicação das decisões e resultados.
