# Reajuste de Preço e Simulação de Margem por Categoria

**Cargo-alvo:** Analista de Pricing / Analista de Precificação / Analista Comercial
**Ferramentas:** Google Sheets (análise exploratória e simulação manual) + Power BI (dashboard interativo com parâmetro What-if e medidas DAX)
**Dataset:** Superstore Sales Dataset
**Link:** https://www.kaggle.com/datasets/vivek468/superstore-dataset-final

---

## Problema de Negócio

A diretoria de Pricing percebeu que a margem líquida vem caindo, apesar do faturamento estável. Não havia clareza se a causa era desconto concedido sem critério pela equipe comercial ou o próprio mix de produto vendido. Este projeto constrói um DRE simplificado por categoria/subcategoria, identifica onde a margem está sendo corroída, e simula o impacto financeiro de uma política de reajuste de preço.

## Premissas

- O dataset não traz custo nem preço unitário diretamente — ambos foram reconstruídos a partir dos dados disponíveis: **Custo Implícito = Receita − Lucro** e **Preço Unitário Implícito = Receita ÷ Quantidade**.
- Critério de "subcategoria crítica": desconto médio acima da média geral do portfólio (16%) **e** margem média abaixo da média geral (12%) — uma regra simples e defensável, não um modelo estatístico complexo.
- A elasticidade de demanda não está disponível no dataset (não há histórico de teste de preço), então foi assumida uma elasticidade conservadora de **0,4** (cada 1% de reajuste no preço reduz o volume em 0,4%) para a simulação principal, com teste de sensibilidade adicional em elasticidade **1,0** para validar o risco.
- O modelo de simulação usa aproximação linear (Volume × (1 − reajuste × elasticidade)), não o modelo multiplicativo/log-log — simplificação aceitável para uma primeira análise, citada como próximo passo.

## Perguntas de Negócio

1. Qual a margem líquida (%) por categoria e subcategoria hoje?
2. Existe correlação entre desconto concedido e queda de margem?
3. Quais subcategorias têm desconto alto e margem baixa ao mesmo tempo — candidatas a revisão de preço?
4. Se reduzíssemos o desconto médio nessas subcategorias críticas, qual seria o ganho de margem?
5. Simulando um reajuste de preço, considerando a queda de volume esperada pela elasticidade assumida, qual o novo cenário de receita e margem?

## Estratégia da Solução

A análise foi conduzida em duas camadas complementares:

**Google Sheets** — construção do raciocínio passo a passo: DRE por categoria/subcategoria via tabela dinâmica, cálculo de correlação estatística entre desconto e margem, identificação formal das subcategorias críticas por regra documentada, e simulação manual de 4 cenários de reajuste (2%, 5%, 8%, 10%) com teste de sensibilidade em duas elasticidades diferentes (0,4 e 1,0).

**Power BI** — tradução da análise validada em ferramenta interativa: medidas DAX organizadas em tabela `_Medidas`, gráfico de dispersão Desconto x Margem (visão do portfólio inteiro), tabela DRE detalhada, e um parâmetro **What-if** que permite simular qualquer percentual de reajuste em tempo real, recalculando a receita simulada apenas nas subcategorias críticas via `CALCULATE` com filtro condicional.

## Insights

- **Correlação de -0,86 entre desconto e margem** — uma relação estatística muito forte, confirmando que a política de desconto é a causa dominante da erosão de margem, não um fator secundário entre vários.
- **6 subcategorias críticas identificadas:** Bookcases, Chairs, Tables, Appliances, Binders e Machines — todas com desconto acima da média (16%) e margem abaixo da média (12%). Em 4 delas (Bookcases, Tables, Appliances, Binders) a margem já é negativa.
- **Desconto alto não é sentença automática de prejuízo:** Copiers (16% de desconto) mantém 32% de margem, e Labels (7% de desconto) chega a 43% — a causa real é a combinação de desconto com produtos de margem intrinsecamente apertada, não o desconto isoladamente.
- **O resultado da simulação depende criticamente da elasticidade real do cliente.** Com elasticidade conservadora (0,4), todo reajuste testado gera ganho. Com elasticidade mais alta (1,0), o mesmo reajuste de 5% inverte para prejuízo de R$ 2.890 — a recomendação só se sustenta se a demanda for de fato pouco sensível a preço nessas categorias.

## Resultado / Impacto Financeiro

Um reajuste de **5%** de preço nas 6 subcategorias críticas recupera um estimado de **R$ 33.519,75** em receita (elasticidade assumida de 0,4). Chairs é o item de maior recuperação isolada (R$ 9.483,54), seguido por Binders (R$ 6.353,33) e Tables (R$ 5.941,37).

Testes de sensibilidade em diferentes percentuais de reajuste (elasticidade 0,4):

| Reajuste | Ganho Total Estimado |
|---|---|
| 2% | R$ 13.685,00 |
| 5% | R$ 33.519,75 |
| 8% | R$ 52.521,98 |
| 10% | R$ 64.727,39 |

## Próximos Passos

- Validar a elasticidade real de demanda via teste controlado (A/B de preço) antes de aplicar reajustes acima de 5%, dado que o modelo se torna desfavorável em cenários de maior sensibilidade a preço.
- Substituir o modelo linear de elasticidade por um modelo multiplicativo/log-log, mais preciso para variações maiores de preço.
- Investigar a causa raiz da margem apertada nas subcategorias críticas além do desconto (custo de fornecedor, mix de produto dentro da subcategoria) antes de decidir entre reajuste de preço ou renegociação de custo.

## Dashboard Power BI

O dashboard interativo traduz a análise validada no Google Sheets em uma ferramenta de simulação em tempo real, com os seguintes componentes:

- **Cartões de indicadores:** Receita Total, Margem %, Desconto Médio e Receita Simulada, atualizados dinamicamente conforme o cenário de reajuste selecionado.
- **Parâmetro What-if (`% Reajuste de Preço`):** slider de -10% a +20% que recalcula a receita simulada em tempo real, aplicando o reajuste apenas nas 6 subcategorias críticas via medida DAX com `CALCULATE` e filtro condicional — as demais subcategorias permanecem no preço original.
- **Gráfico de dispersão (Desconto Médio x Margem %):** visão do portfólio completo, com o tamanho da bolha representando a receita de cada subcategoria — evidencia visualmente o quadrante crítico (desconto alto, margem baixa).
- **Gráfico de barras (Ganho/Perda por Subcategoria):** mostra a receita adicional estimada em cada uma das 6 subcategorias críticas, reagindo ao slider de reajuste.
- **Tabela detalhada (DRE por Categoria/Subcategoria):** Receita, Custo, Lucro, Desconto Médio e Margem % por subcategoria, espelhando o DRE construído no Sheets.

🔗 [Acessar dashboard interativo no Power BI](https://app.powerbi.com/view?r=eyJrIjoiM2M1YT1lNjctODk4My00ODY3LTg4MTMtMjhhMzN1M2Y2NWJkIiwidCI6ImE0NTMyMzQyLWRjNjktNDhjMC1iODJhLTRhMWQ1ZDg2NGU2YiJ9)
