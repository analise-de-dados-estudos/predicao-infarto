# 00 — Visão geral do manual

Este manual descreve, do início ao fim, como executar o projeto de **predição de
doença cardíaca / risco de infarto**. Cada documento cobre uma etapa, com o que
fazer, como decidir e uma checklist de conclusão.

## As duas bases do projeto

| Base | Natureza | Papel |
|---|---|---|
| `heart.csv` (fedesoriano) — 918 × 12 | **Real** (5 bases clínicas da UCI) | **Base principal.** Toda a análise e a modelagem acontecem aqui |
| `heart_attack_prediction_dataset.csv` — 8.763 × 26 | Sintética | Contraprova. Mesmo pipeline, para comparar resultados na etapa 12 |

Trabalhe as etapas 01 a 10 **na base principal**. A base sintética só volta ao
jogo na etapa 12.

## Mapa das etapas

| # | Documento | Entrega da etapa | Branch sugerida |
|---|---|---|---|
| 01 | [Entendimento do problema](01_entendimento_do_problema.md) | Pergunta de negócio, hipóteses e métrica definidas | `feature/definicao-escopo` |
| 02 | [Coleta e fontes de dados](02_coleta_e_fontes_de_dados.md) | Bases em `dados_brutos/` + origem documentada | `feature/coleta-dados` |
| 03 | [Dicionário de variáveis](03_dicionario_de_variaveis.md) | `dados_tratados/dicionario_variaveis.csv` | `feature/dicionario-dados` |
| 04 | [Limpeza e tratamento](04_limpeza_e_tratamento.md) | `dados_tratados/base_analitica.csv` | `feature/limpeza-dados` |
| 05 | [Análise exploratória (EDA)](05_analise_exploratoria.md) | Notebook + gráficos em `resultados/figuras/` | `feature/analise-exploratoria` |
| 06 | [Engenharia de atributos](06_engenharia_de_atributos.md) | `dados_tratados/base_modelavel.csv` | `feature/engenharia-atributos` |
| 07 | [Modelagem](07_modelagem.md) | Modelos treinados em `resultados/modelos/` | `feature/modelagem` |
| 08 | [Avaliação e métricas](08_avaliacao_e_metricas.md) | Tabela comparativa em `resultados/metricas/` | `feature/avaliacao-modelos` |
| 09 | [Interpretabilidade](09_interpretabilidade.md) | Gráficos de importância de variáveis | `feature/interpretabilidade` |
| 12 | [Comparação entre bases](12_comparacao_entre_bases.md) | Tabela real × sintética | `feature/comparacao-bases` |
| 10 | [Conclusão e entrega](10_conclusao_e_entrega.md) | Relatório final + README atualizado | `release/v1.0.0` |
| 11 | [Git Flow e padrão de commit](11_git_flow_e_padrao_de_commit.md) | — (consulta contínua) | — |

> A etapa 12 é executada **antes** da 10 — ela produz um dos resultados que
> entram no relatório final.

## Fluxo do dado dentro do projeto

```
Kaggle (fedesoriano)
  │  download
  ▼
dados_brutos/heart.csv                               ← imutável
  │  limpeza + padronização (etapa 04)
  ▼
dados_tratados/base_analitica.csv                    ← legível, para EDA
  │  encoding + escala + seleção (etapa 06)
  ▼
dados_tratados/base_modelavel.csv                    ← numérica, para o modelo
  │  treino/teste (etapa 07)
  ▼
resultados/modelos/  +  resultados/metricas/  +  resultados/figuras/
  │  mesmo pipeline na base sintética (etapa 12)
  ▼
Relatório final (etapa 10)
```

## Como usar este manual

- **Uma etapa = uma branch = um Pull Request.** Não pule para modelagem antes de
  fechar a limpeza; o retrabalho é caro.
- Ao terminar cada etapa, marque a checklist do documento e registre no
  `logs/` as decisões que você tomou (o que descartou e por quê). Isso vira o
  texto do relatório final praticamente pronto.
- Onde o manual diz "decida", não existe resposta única — o que conta é
  **registrar a justificativa**.

## Estimativa de esforço

| Etapa | Peso relativo | Observação |
|---|---|---|
| 01–03 | 15% | Parece burocracia, evita 80% dos erros |
| 04 | 20% | Nesta base o ponto crítico é o colesterol zerado |
| 05 | 20% | Onde saem as melhores conclusões do relatório |
| 06–07 | 20% | Rápido se as etapas anteriores foram bem feitas |
| 08–09 | 10% | — |
| 12 | 5% | Reaproveita todo o pipeline já pronto |
| 10 | 10% | Não deixe para a última hora |
