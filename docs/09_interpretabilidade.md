# 09 — Interpretabilidade do modelo

> **Objetivo da etapa:** explicar **por que** o modelo decide o que decide.
> Em saúde, um modelo que não se explica não é usado.

---

## 9.1 Por que esta etapa existe

Um modelo que aponta "doença cardíaca" sem justificativa é inútil na prática
clínica. A interpretabilidade responde a três perguntas:

1. Quais variáveis mais pesam na decisão?
2. Em que direção cada uma empurra (aumenta ou reduz a probabilidade)?
3. Por que **este paciente específico** foi classificado assim?

Aqui a base real dá o que a sintética não daria: um ranking **estável** e
**clinicamente coerente**. É a etapa que valida o trabalho inteiro.

## 9.2 Técnicas por tipo de modelo

| Modelo | Técnica | O que entrega |
|---|---|---|
| Regressão Logística | Coeficientes e razão de chances (odds ratio) | Direção e magnitude do efeito de cada variável |
| Árvore de Decisão | Visualização da árvore (`plot_tree`) | Regras legíveis: "se ST_Slope = Flat e Oldpeak > 1,5, então…" |
| Random Forest / Boosting | `feature_importances_` | Ranking de importância (sem direção) |
| Qualquer modelo | **Permutation importance** | Queda de desempenho ao embaralhar cada variável — mais confiável |
| Qualquer modelo | **SHAP** | Contribuição de cada variável, global e por paciente |

## 9.3 Importância global

Gere e comente:

- **Gráfico de barras** com o top 10 de variáveis mais importantes
- **Permutation importance** com barra de erro (mais robusta que a importância
  nativa, que favorece variáveis contínuas e de alta cardinalidade)
- **SHAP summary plot** (beeswarm), que mostra importância **e** direção

### O que esperar

O topo do ranking costuma ser ocupado por:

| Variável | Por quê |
|---|---|
| `ST_Slope` | Inclinação do segmento ST — o achado mais discriminante do teste de esforço |
| `ChestPainType` (em especial `ASY`) | Tipo de dor no peito |
| `Oldpeak` | Depressão do segmento ST |
| `ExerciseAngina` | Angina induzida por exercício |
| `MaxHR` | Frequência cardíaca máxima atingida |

E na base: `Cholesterol` e `RestingBP` costumam ficar no fim do ranking.

**Perguntas a responder no texto:**

- O ranking faz sentido clínico? Aqui faz: quatro das cinco variáveis mais
  importantes vêm do teste ergométrico, que é justamente o exame usado para
  detectar isquemia. **Escreva isso no relatório** — é a evidência de que o
  modelo aprendeu o fenômeno, e não ruído
- Alguma variável suspeita aparece no topo? Se `colesterol_ausente` (a
  indicadora que você criou) ficar muito alta, é sinal de que a ausência do
  exame está correlacionada com a origem hospitalar do registro — discussão
  excelente sobre viés de coleta
- O ranking é **estável**? Ver 9.6

## 9.4 Interpretação da regressão logística

A regressão logística permite a leitura mais direta. Para cada variável:

- Coeficiente **positivo** → aumenta a chance de doença
- Coeficiente **negativo** → reduz
- `exp(coeficiente)` = **razão de chances**: quantas vezes a chance se
  multiplica a cada unidade da variável

Exemplo de redação: *"pacientes com `ST_Slope` plana têm chance X vezes maior de
apresentar doença coronariana do que aqueles com inclinação ascendente,
mantidas as demais variáveis constantes"*.

⚠️ Só interprete coeficientes se as variáveis estiverem padronizadas ou se você
declarar a unidade. E lembre: **associação não é causalidade** — `MaxHR` baixa
não *causa* doença, é consequência da limitação funcional que a doença provoca.

## 9.5 Interpretação individual (SHAP local)

Escolha 3 pacientes do conjunto de teste e apresente:

- Um caso classificado como doente **corretamente**
- Um **falso negativo** (o erro mais caro) — por que o modelo errou?
- Um **falso positivo**

Para cada um, o *waterfall plot* do SHAP mostra quais variáveis empurraram a
previsão para cima ou para baixo. É o material mais persuasivo do relatório, e
a análise dos erros é o que mostra profundidade.

## 9.6 Teste de estabilidade

Rode o modelo com 5 sementes diferentes e compare o top 5 de variáveis.

| Resultado | Interpretação |
|---|---|
| Top 5 praticamente idêntico | ✅ O modelo encontrou estrutura real nos dados |
| Ranking muda a cada rodada | ⚠️ Instabilidade — comum em base pequena; reporte a média |

Espere estabilidade aqui. Guarde este resultado: na etapa 12 você mostra que na
base sintética o ranking muda a cada rodada, e o contraste fecha o argumento.

## 9.7 Comunicação dos achados

- Traduza nomes técnicos nos gráficos (`ST_Slope` → "Inclinação do segmento ST")
- Escreva uma frase de conclusão por gráfico
- Separe claramente **o que o modelo aprendeu** de **o que é verdade clínica
  estabelecida**. O modelo encontrou associações nesta amostra de ~918
  pacientes, coletada em quatro países entre as décadas de 1980 e 1990 — não é
  uma descoberta médica nova

## ✅ Checklist da etapa

- [ ] Importância global calculada (nativa + permutation)
- [ ] Top 10 de variáveis em gráfico exportado
- [ ] Coeficientes da regressão logística interpretados com razão de chances
- [ ] SHAP aplicado (global e 3 casos individuais, incluindo um erro)
- [ ] Estabilidade do ranking testada com 5 sementes
- [ ] Plausibilidade clínica discutida por escrito
- [ ] Figuras salvas em `resultados/figuras/`

```bash
git checkout -b feature/interpretabilidade develop
git commit -m "feat(interpretacao): adiciona importancia de variaveis e permutation importance"
git commit -m "feat(interpretacao): adiciona analise SHAP global e casos individuais"
git commit -m "docs(interpretacao): discute plausibilidade clinica do ranking"
```

➡️ Próxima: [12 — Comparação entre bases](12_comparacao_entre_bases.md)
