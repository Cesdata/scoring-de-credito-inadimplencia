# Scoring de Crédito — Previsão de Inadimplência

Modelo preditivo que estima a probabilidade de inadimplência em cobranças mensais de clientes, desenvolvido como case técnico de ciência de dados aplicado a risco de crédito.

**AUC (validação): 0.94 | Brier Score (validação): 0.035**

## Contexto do problema

O cenário é o de uma operação de crédito que acompanha transações financeiras de clientes ao longo do tempo, com o objetivo de apoiar decisões proativas de cobrança: identificar, com antecedência, quais cobranças têm maior probabilidade de serem pagas com atraso (5 dias ou mais em relação ao vencimento), para que ações preventivas possam reduzir o índice de inadimplência.

O problema é tratado como uma estimativa de **probabilidade contínua** (entre 0 e 1) por cobrança, não como uma classificação binária.

## Dados

O projeto utiliza quatro bases relacionadas por `ID_CLIENTE` e `SAFRA_REF`:

| Base | Granularidade | Conteúdo |
|---|---|---|
| `base_cadastral` | 1 linha por cliente | Dados cadastrais: porte, segmento, CEP, e-mail, data de cadastro |
| `base_info` | 1 linha por cliente/mês | Renda do mês anterior e número de funcionários |
| `base_pagamentos_desenvolvimento` | 1 linha por cobrança | Histórico de cobranças com data de pagamento conhecida (usada para treino) |
| `base_pagamentos_teste` | 1 linha por cobrança | Cobranças mais recentes, sem data de pagamento (usada para gerar as previsões finais) |

As saídas de cada etapa (tabelas, métricas, gráficos) estão preservadas no próprio notebook.

## Metodologia

O notebook está organizado nas seguintes etapas:

1. **Subida e análise das bases:** leitura das 4 bases, verificação da proporção de valores nulos e da duplicidade/granularidade entre elas, confirmando o relacionamento entre as tabelas.
2. **Construção da Target:** conversão das datas, cálculo do atraso (`data de pagamento − data de vencimento`) e criação da coluna binária de inadimplência.
3. **Merge das bases:** união das bases cadastral e mensal com a base de pagamentos, verificação e limpeza de nulos, e criação da feature de tempo de cadastro do cliente.
4. **Ordenação do histórico por cliente:** ordenação cronológica das cobranças, cálculo das features de histórico considerando apenas cobranças com data anterior à cobrança avaliada (evitando vazamento temporal), e decisão sobre o tratamento dos valores nulos gerados na primeira cobrança de cada cliente.
5. **Modelagem:** definição da proporção treino/validação, escolha justificada do modelo e treinamento.
6. **Validação (treino):** cálculo da probabilidade de cada cobrança e avaliação do modelo no conjunto de validação.
7. **Validação no teste:** preparo da base de teste com os mesmos tratamentos aplicados à base de desenvolvimento, e treinamento do modelo final.
8. **Previsões:** geração das probabilidades finais e do arquivo de submissão.
9. **Versões utilizadas:** registro do ambiente (Python e bibliotecas).

## Principais decisões técnicas

- **Prevenção de vazamento temporal**: cada feature de histórico do cliente é calculada usando somente as cobranças anteriores dele, nunca a própria cobrança que está sendo prevista, nunca cobranças futuras.
- **Validação temporal, não aleatória**: o corte treino/validação respeita a ordem cronológica das safras, simulando como o modelo seria usado em produção.
- **Qualidade de dado**: identificação de registros com datas de vencimento inconsistentes (erros de digitação), removidos do treinamento por comprometerem a confiabilidade do rótulo.
- **Escolha do modelo**: preferência por um classificador que lida nativamente com valores ausentes e variáveis categóricas, evitando imputações artificiais que poderiam introduzir vazamento sutil.
- **Foco em probabilidade, não apenas classificação**: avaliação explícita da calibração das previsões (Brier Score), não apenas da capacidade de ordenação (AUC).

## Resultados

| Métrica | Valor (validação) |
|---|---|
| AUC-ROC | 0.94 |
| Brier Score (modelo) | 0.035 |
| Brier Score (baseline ingênuo) | 0.059 |

## Tecnologias

- Python 3.12.13
- pandas 2.2.2
- numpy 2.0.2
- scikit-learn 1.6.1
- matplotlib 3.10.0

Versões completas em [`requirements.txt`](./requirements.txt).

## Conteúdo do repositório

- [`Notebook_score.ipynb`](./Notebook_score.ipynb) — notebook completo, com código, comentários e saídas de cada etapa já executadas.
- [`probab_inadimplencia.csv`](./probab_inadimplencia.csv) — previsões finais de probabilidade de inadimplência geradas pelo modelo.
- [`requirements.txt`](./requirements.txt) — bibliotecas e versões utilizadas.

## Próximos passos

- Calibração explícita das probabilidades (ex: `CalibratedClassifierCV`) e comparação com a calibração natural do modelo.
- Tunagem de hiperparâmetros com validação cruzada temporal (múltiplas janelas).
- Tratamento na origem dos registros com data de vencimento inconsistente, em vez de apenas removê-los.
- Exploração mais rica de variáveis geográficas (CEP, DDD), que se mostraram relevantes na importância de features do modelo.
