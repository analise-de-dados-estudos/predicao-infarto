# notebooks/

Notebooks numerados **na ordem de execução**. Quem clonar o repositório deve
conseguir rodar de `01` a `07`, em sequência, e reproduzir todos os resultados.

| Notebook | Etapa do manual | Entrada | Saída |
|---|---|---|---|
| `01_exploracao_inicial.ipynb` | [02](../docs/02_coleta_e_fontes_de_dados.md) / [03](../docs/03_dicionario_de_variaveis.md) | `dados_brutos/heart.csv` | Diagnóstico da base, dicionário |
| `02_limpeza_tratamento.ipynb` | [04](../docs/04_limpeza_e_tratamento.md) | `dados_brutos/heart.csv` | `dados_tratados/base_analitica.csv` |
| `03_analise_exploratoria.ipynb` | [05](../docs/05_analise_exploratoria.md) | `base_analitica.csv` | `resultados/figuras/` |
| `04_engenharia_atributos.ipynb` | [06](../docs/06_engenharia_de_atributos.md) | `base_analitica.csv` | `dados_tratados/base_modelavel.csv` |
| `05_modelagem.ipynb` | [07](../docs/07_modelagem.md) | `base_modelavel.csv` | `resultados/modelos/` |
| `06_avaliacao_interpretacao.ipynb` | [08](../docs/08_avaliacao_e_metricas.md) / [09](../docs/09_interpretabilidade.md) | modelos treinados | `resultados/metricas/`, `resultados/figuras/` |
| `07_comparacao_bases.ipynb` | [12](../docs/12_comparacao_entre_bases.md) | as duas bases brutas | Figuras e tabela comparativa |

## Boas práticas

- **Caminhos relativos**, sempre (`../dados_brutos/heart.csv`). Nunca
  `C:\Users\seu-nome\...`
- Primeira célula: título, objetivo do notebook e data
- Célula de markdown antes de cada bloco, explicando o que vem a seguir
- Comentário depois de cada gráfico dizendo **o que ele mostra**
- `Kernel → Restart & Run All` antes de commitar: se quebrar, está fora de ordem
- `random_state` fixo em tudo que sorteia
- Pré-processamento dentro de `Pipeline`, para não vazar informação do teste
