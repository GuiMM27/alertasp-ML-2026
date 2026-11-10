# Uso responsável de IA como copiloto — ALERTASP

Este arquivo registra o emprego de assistentes de IA durante a GS. Não são apresentadas instruções fabricadas nem números de experimento sintéticos.

## Solicitações reais feitas pela equipe (trechos da conversa)

- **Planejamento da rubrica:** “Atue como cientista de dados/engenheiro de ML e orientador acadêmico… cumprir integralmente a rubrica da FIAP.” Foi solicitado usar o PDF oficial, dados reais públicos e separar treino/teste corretamente.
- **Revisão do objetivo do produto:** “Com base na chuva observada até o início de um dia e nas características históricas de determinada região de São Paulo, qual é a probabilidade estimada de haver pelo menos um registro de alagamento ou inundação nessa região durante as próximas 24 horas?”
- **Restrições da coleta:** “Vamos fazer so da capital… te mandei o de 2013 de sao paulo… me mande o drive inteiro…” O escopo foi limitado à cidade e à estação INMET A701 São Paulo–Mirante.
- **Pedido de implementação final:** “vou te mandar tudo oq temos e me mande o modelo completo e pronto para entrega seguindo tudo que ja combinamos”.

## Trabalho humano e revisão

A equipe obteve os CSVs originais de 2013–2026 e a série histórica GeoSampa, definiu o problema CitizenTech, executou discussões sobre a utilidade ao cidadão e revisou outputs. A IA auxiliou com exploração de dados, scripts, documentação, validação e formulação das limitações. **As métricas do notebook foram realmente calculadas a partir dos arquivos fornecidos**; os resultados não foram inventados. Conferir e entender código, fontes, arquivos e conclusões antes da submissão.

## Decisões corretivas documentadas

- Separação da pergunta de classificação de tipo (experimento anterior) e previsão de **registro municipal** (experimento desta entrega).
- Conversão UTC para dia local e prevenção de vazamento por uso de chuva apenas anterior ao dia previsto.
- Ausências meteorológicas mantidas como desconhecidas; janelas 3/7 dias calculadas apenas com cobertura mínima, acompanhadas de frações de cobertura.
- Separação temporal desenvolvimento 2013–2022, calibração 2023, verificação 2024, avaliação retrospectiva 2025, suplementar 2026.
- Conclusão restrita: o RF teve AP maior que baseline mensal em 2025, mas calibração Brier pior; **não se apresenta a porcentagem como risco operacional de alagamento**.
