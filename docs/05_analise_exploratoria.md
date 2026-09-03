# 05 — Análise exploratória (EDA)

> **Objetivo da etapa:** responder às perguntas e hipóteses da etapa 01 com
> números e gráficos, e entender o comportamento das variáveis antes de modelar.
>
> **Notebook:** `notebooks/03_analise_exploratoria.ipynb`
> **Entrada:** `dados_tratados/base_analitica.csv`
> **Saída:** gráficos em `resultados/figuras/` + tabelas em `resultados/tabelas/`

---

## 5.1 Princípio da etapa

Todo gráfico precisa responder a uma pergunta escrita antes de ser feito. Gráfico
sem pergunta é enfeite e não entra no relatório.

Estrutura de cada bloco do notebook:

```
Pergunta → Gráfico/tabela → Leitura em 1-3 frases → Implicação para a modelagem
```

## 5.2 Análise univariada

### Variável alvo (sempre primeiro)

- Contagem e percentual de `doenca_cardiaca` — esperado: ~55% / ~45%
- **Conclusão a registrar:** base quase balanceada → não precisa de SMOTE, e a
  acurácia engana menos que o normal (mas ainda não é a métrica principal)

### Numéricas

Para `idade`, `pressao_repouso`, `colesterol`, `freq_cardiaca_max` e
`depressao_st`:

- Histograma + boxplot
- Média, mediana, desvio padrão, mínimo, máximo, assimetria
- Pergunta-guia: a distribuição é normal, assimétrica ou uniforme?

O que esperar em dados clínicos reais:

| Variável | Comportamento esperado |
|---|---|
| `idade` | Aproximadamente normal, concentrada entre 50 e 60 anos |
| `colesterol` | Assimétrica à direita, com cauda longa (após tratar os zeros) |
| `pressao_repouso` | Aproximadamente normal |
| `freq_cardiaca_max` | Aproximadamente normal, com relação inversa à idade |
| `depressao_st` | Fortemente concentrada em zero, com cauda à direita |

> Compare com a base sintética da etapa 12: lá, todas as contínuas são
> **uniformes**. O contraste entre os dois conjuntos de histogramas é uma das
> figuras mais fortes do relatório.

### Categóricas

Para `sexo`, `tipo_dor_peito`, `ecg_repouso`, `angina_exercicio`,
`inclinacao_st` e `glicemia_jejum_alta`:

- Gráfico de barras com contagem e percentual
- Verificar categorias raras (< 5%) — `TA` e `Down` costumam ser pouco
  frequentes; considere agrupar se atrapalharem o modelo
- Registrar o desbalanceamento de `sexo` (bem mais homens) — é uma limitação a
  declarar

## 5.3 Análise bivariada — cada variável contra o alvo

É o núcleo da EDA e o que testa as hipóteses da etapa 01.

| Tipo de variável | Visualização | Teste estatístico |
|---|---|---|
| Numérica × alvo | Boxplot ou violino por classe; histogramas sobrepostos | Teste t (normal) ou Mann-Whitney (não normal) |
| Categórica × alvo | Barras empilhadas 100% ou agrupadas | Qui-quadrado |
| Binária × alvo | Tabela cruzada com percentual por linha | Qui-quadrado |

**Sempre em percentual por grupo**, nunca em contagem absoluta.

### Roteiro mínimo (ligado às hipóteses)

| Hipótese | Análise | O que costuma aparecer |
|---|---|---|
| H1 — idade | Taxa de doença por faixa etária | Cresce com a idade |
| H2 — sexo | Tabela cruzada sexo × alvo | Proporção maior entre homens |
| H3 — angina de exercício | Tabela cruzada + qui-quadrado | Associação forte |
| H4 — freq. cardíaca máxima | Boxplot por classe | **Menor** no grupo com doença |
| H5 — inclinação ST | Barras empilhadas 100% | `Flat`/`Down` muito associados a doença |
| H6 — colesterol | Boxplot + correlação | Discriminação fraca |

Para cada uma, escreva o veredito: **confirmada / refutada / inconclusiva**.

> Não copie as expectativas da coluna da direita para o relatório. Elas servem
> para você perceber quando um resultado está estranho — se `MaxHR` sair *maior*
> no grupo com doença, provavelmente há erro no código.

### Duas análises que rendem bons parágrafos

1. **`ChestPainType = ASY` (assintomático) tem a maior proporção de doença.**
   Contraintuitivo: o grupo sem dor no peito é o mais doente. Explicação
   provável: são pacientes encaminhados à angiografia por outros indícios.
   Discuta o **viés de seleção** — é análise crítica de verdade.
2. **Colesterol discrimina pouco**, contrariando o senso comum. Some a isso o
   fato de 19% dos valores serem ausentes disfarçados de zero e você tem uma
   discussão completa sobre qualidade de dado.

## 5.4 Análise multivariada

- **Matriz de correlação** (Pearson) das numéricas + heatmap
  - Objetivo 1: quais variáveis se correlacionam com o alvo
  - Objetivo 2: quais se correlacionam entre si (multicolinearidade)
  - Esperado: `depressao_st` e `freq_cardiaca_max` entre as mais correlacionadas
    com o alvo; `idade` × `freq_cardiaca_max` com correlação negativa entre si
- **Para categóricas**, correlação de Pearson não se aplica. Use qui-quadrado ou
  V de Cramér — as variáveis mais fortes desta base (`ST_Slope`,
  `ChestPainType`) são categóricas e ficariam invisíveis no heatmap
- Recortes cruzados: taxa de doença por faixa etária **e** sexo; por tipo de dor
  **e** angina de exercício

## 5.5 Cuidados de interpretação

- **Correlação não é causalidade.** Angina de exercício associada a doença não
  significa que o exercício cause a doença — a angina é sintoma, não causa.
- **Correlação linear ~0 não significa ausência de relação.** Pode haver relação
  não linear (teste com boxplots por faixa) ou condicional (só em um subgrupo).
- **Cuidado com o viés da amostra.** São pacientes que já foram encaminhados
  para angiografia — não é a população geral. As proporções encontradas aqui não
  valem para a população brasileira.

## 5.6 Padrão dos gráficos

- Título que diz a conclusão, não o tipo do gráfico
  - ❌ "Boxplot de MaxHR" ✅ "Frequência cardíaca máxima é menor em pacientes com doença"
- Eixos rotulados com unidade
- Categorias com rótulo legível (`Assintomatico`, não `ASY`)
- Paleta consistente; cor com significado fixo (classe 1 sempre na mesma cor)
- Exportar em PNG (150+ dpi) para `resultados/figuras/`, com nome descritivo:
  `05_taxa_doenca_por_faixa_etaria.png`

## 5.7 Entregável da etapa

Uma seção "Principais achados" com 5 a 10 conclusões numeradas, cada uma
apontando o gráfico que a sustenta. Esse texto é o esqueleto do relatório final.

## ✅ Checklist da etapa

- [ ] Distribuição do alvo analisada
- [ ] Todas as numéricas com histograma + boxplot
- [ ] Todas as categóricas com gráfico de frequência
- [ ] Todas as variáveis cruzadas com o alvo
- [ ] Seis hipóteses da etapa 01 respondidas com veredito
- [ ] Matriz de correlação gerada e comentada
- [ ] Associação das categóricas medida (qui-quadrado / V de Cramér)
- [ ] Multicolinearidade identificada
- [ ] Figuras exportadas para `resultados/figuras/`
- [ ] Seção "Principais achados" escrita

```bash
git checkout -b feature/analise-exploratoria develop
git commit -m "feat(eda): adiciona analise univariada das variaveis numericas"
git commit -m "feat(eda): adiciona matriz de correlacao e heatmap"
git commit -m "feat(eda): testa hipoteses H1 a H6 com graficos e testes estatisticos"
git commit -m "docs(eda): registra principais achados da analise exploratoria"
```

➡️ Próxima: [06 — Engenharia de atributos](06_engenharia_de_atributos.md)
