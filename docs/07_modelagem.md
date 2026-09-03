# 07 — Modelagem

> **Objetivo da etapa:** treinar modelos de classificação, do mais simples ao
> mais complexo, e escolher o melhor com critério.
>
> **Notebook:** `notebooks/05_modelagem.ipynb`
> **Entrada:** `dados_tratados/base_modelavel.csv`
> **Saída:** modelos em `resultados/modelos/` + métricas em `resultados/metricas/`

---

## 7.1 Estratégia: do simples ao complexo

Nunca comece pelo modelo mais sofisticado. A ordem serve para saber se a
complexidade adicional está pagando.

```
1. Baseline trivial      -> prever sempre a classe majoritária
2. Regressão logística   -> modelo linear, interpretável
3. Árvore de decisão     -> não linear, visualizável
4. Random Forest         -> ensemble robusto
5. Gradient Boosting     -> costuma ser o melhor em dado tabular
6. (opcional) KNN / SVM  -> comparação
```

**Regra:** se o modelo do passo 5 não superar o do passo 2 de forma
consistente, entregue o do passo 2. Simplicidade é qualidade, não preguiça — e
nesta base a regressão logística costuma ficar a menos de 2 pontos percentuais
de AUC do melhor modelo, com a vantagem de ser totalmente interpretável.

## 7.2 Baseline — obrigatório

| Baseline | O que faz | Resultado esperado |
|---|---|---|
| `DummyClassifier(strategy='most_frequent')` | Responde sempre "com doença" (classe majoritária) | Acurácia ~55%, ROC AUC = 0,50 |

Toda métrica dos modelos seguintes é comparada contra esta linha.

## 7.3 Modelos e configuração inicial

| Modelo | Hiperparâmetros iniciais | Escalonar? | Interpretável? |
|---|---|---|---|
| **Regressão Logística** | `max_iter=1000` | ✅ Sim | ✅ Alta (coeficientes) |
| **Árvore de Decisão** | `max_depth=4`, `min_samples_leaf=20` | ❌ Não | ✅ Alta (visualizável) |
| **Random Forest** | `n_estimators=300`, `max_depth=8`, `min_samples_leaf=5` | ❌ Não | 🟡 Média (importâncias) |
| **Gradient Boosting / XGBoost** | `n_estimators=200`, `learning_rate=0.05`, `max_depth=3` | ❌ Não | 🟡 Média (com SHAP) |
| **KNN** | `n_neighbors=15` | ✅ Sim | ❌ Baixa |
| **SVM** | `kernel='rbf'`, `probability=True` | ✅ Sim | ❌ Baixa |

> Com 918 linhas, modelos profundos superajustam rápido. Comece com árvores
> rasas (`max_depth` 3 a 8) e `min_samples_leaf` alto.

Use `Pipeline` para encadear imputação + escalonamento + modelo: garante que
tudo seja ajustado apenas no treino em cada fold.

## 7.4 Validação cruzada

Não confie em uma única divisão treino/teste — com ~184 registros no teste, a
métrica oscila muito conforme o sorteio.

- **`StratifiedKFold`, 5 ou 10 folds**, embaralhado, com `random_state` fixo
- Reporte **média ± desvio padrão** de cada métrica
- Desvio alto entre folds = modelo instável

Um resultado como `ROC AUC = 0,91 ± 0,02` diz muito mais que um `0,93` isolado.

## 7.5 Otimização de hiperparâmetros

| Método | Quando usar |
|---|---|
| `GridSearchCV` | Poucos parâmetros — viável aqui, a base é pequena |
| `RandomizedSearchCV` | Espaço grande |
| Ajuste manual | Só para entender o efeito de um parâmetro |

Configure `scoring='roc_auc'` e `cv=StratifiedKFold(5)`. Registre o melhor
conjunto de parâmetros encontrado.

Ganho típico da otimização nesta base: 1 a 3 pontos de AUC. Se der muito mais
que isso, desconfie de vazamento.

## 7.6 Desempenho esperado

Valores de referência para você saber se está no caminho certo:

| Modelo | ROC AUC esperado | Acurácia esperada |
|---|---|---|
| Baseline | 0,50 | ~0,55 |
| Regressão Logística | 0,88 – 0,92 | 0,84 – 0,88 |
| Árvore de Decisão | 0,82 – 0,88 | 0,78 – 0,85 |
| Random Forest | 0,90 – 0,93 | 0,85 – 0,90 |
| Gradient Boosting | 0,90 – 0,94 | 0,85 – 0,90 |

**Como ler estes números:**

| Se o seu resultado for | Provável causa |
|---|---|
| Muito **abaixo** (AUC < 0,80) | Erro de encoding, escalonamento ou variável importante descartada |
| Dentro da faixa | ✅ Está correto |
| **Acima de 0,97** | 🚨 Vazamento de dados. Verifique: o alvo entrou nas preditoras? Escalonou antes de dividir? Há duplicatas? |

O terceiro caso é o mais perigoso, porque parece uma vitória. Sempre investigue
métrica alta demais.

## 7.7 Overfitting — como detectar

| Sinal | Interpretação |
|---|---|
| Treino 0,99 / Teste 0,85 | Overfitting — reduza a profundidade |
| Treino 0,92 / Teste 0,90 | ✅ Saudável |
| Grande variação entre folds | Instabilidade — normal em base pequena, mas monitore |
| Métrica muda muito ao trocar `random_state` | Reporte média de várias sementes |

Como reduzir: limitar `max_depth`, aumentar `min_samples_leaf`, usar
regularização (`C` menor na logística), reduzir número de variáveis.

## 7.8 Registro dos experimentos

Mantenha `resultados/metricas/comparativo_modelos.csv`:

| modelo | parametros | acuracia | precisao | recall | f1 | roc_auc | desvio_auc | data |
|---|---|---|---|---|---|---|---|---|

Salve cada modelo treinado em `resultados/modelos/` com `joblib`, com nome
descritivo: `random_forest_v1.joblib`.

## ✅ Checklist da etapa

- [ ] Baseline trivial executado e registrado
- [ ] Ao menos 4 modelos treinados
- [ ] Todos com validação cruzada estratificada
- [ ] `random_state` fixo em todos
- [ ] Pipeline usado (sem vazamento no pré-processamento)
- [ ] Hiperparâmetros otimizados no melhor candidato
- [ ] Overfitting verificado (treino × teste)
- [ ] Métrica dentro da faixa esperada — ou a divergência investigada
- [ ] Tabela comparativa preenchida
- [ ] Modelos serializados em `resultados/modelos/`

```bash
git checkout -b feature/modelagem develop
git commit -m "feat(modelo): adiciona baseline com DummyClassifier"
git commit -m "feat(modelo): treina regressao logistica com validacao cruzada"
git commit -m "feat(modelo): treina random forest e gradient boosting"
git commit -m "feat(modelo): otimiza hiperparametros com GridSearchCV"
```

➡️ Próxima: [08 — Avaliação e métricas](08_avaliacao_e_metricas.md)
