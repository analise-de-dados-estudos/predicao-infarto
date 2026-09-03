# 08 — Avaliação e métricas

> **Objetivo da etapa:** medir corretamente o desempenho dos modelos, escolher o
> melhor e interpretar o que os números querem dizer.
>
> **Notebook:** `notebooks/06_avaliacao_interpretacao.ipynb`

---

## 8.1 Matriz de confusão

Ponto de partida de toda avaliação.

|  | **Previsto: sem doença** | **Previsto: com doença** |
|---|---|---|
| **Real: sem doença** | Verdadeiro Negativo (VN) | Falso Positivo (FP) |
| **Real: com doença** | **Falso Negativo (FN)** ⚠️ | Verdadeiro Positivo (VP) |

Neste projeto:

- **Falso Negativo** = paciente doente classificado como saudável → **erro mais
  grave**, é o paciente que deixa de ser investigado
- **Falso Positivo** = paciente saudável classificado como doente → custo de um
  exame a mais; incômodo, não tragédia

Portanto: **maximizar recall da classe 1**, aceitando perda de precisão dentro
de um limite razoável.

## 8.2 As métricas

| Métrica | Fórmula | O que responde | Peso aqui |
|---|---|---|---|
| **Acurácia** | (VP+VN)/total | Quanto acertei no geral | 🟡 Médio — a base é quase balanceada, então engana menos |
| **Precisão** | VP/(VP+FP) | Dos que apontei como doentes, quantos eram | 🟡 Médio |
| **Recall / Sensibilidade** | VP/(VP+FN) | Dos doentes, quantos encontrei | 🔴 **Alto** |
| **Especificidade** | VN/(VN+FP) | Dos saudáveis, quantos identifiquei | 🟡 Médio |
| **F1-score** | 2·(P·R)/(P+R) | Equilíbrio precisão × recall | 🟡 Médio |
| **ROC AUC** | área sob a curva ROC | Capacidade de separar as classes | 🔴 **Principal** |
| **PR AUC** | área sob precisão-recall | Alternativa em desbalanceamento forte | 🟢 Baixo aqui |

### Como ler o ROC AUC

| Valor | Leitura |
|---|---|
| 0,50 | Igual a jogar uma moeda — o modelo não aprendeu nada |
| 0,60 – 0,70 | Fraco |
| 0,70 – 0,80 | Aceitável |
| 0,80 – 0,90 | Bom — **faixa esperada neste projeto** |
| 0,90 – 0,95 | Muito bom — atingível aqui com ensemble |
| > 0,97 | 🚨 Desconfie de vazamento de dados |

## 8.3 Meta do projeto

| Métrica | Meta |
|---|---|
| ROC AUC | ≥ 0,85 |
| Recall (classe 1) | ≥ 0,85 |
| Precisão (classe 1) | ≥ 0,80 |

Se bater as três, o modelo está pronto para o relatório.

## 8.4 Curvas a gerar

| Curva | O que mostra | Onde salvar |
|---|---|---|
| **ROC** | Taxa de VP × taxa de FP, para todos os limiares | `resultados/figuras/08_roc_comparativa.png` |
| **Precisão-Recall** | Trade-off entre as duas | `resultados/figuras/08_pr_<modelo>.png` |
| **Curva de calibração** | Se a probabilidade prevista corresponde à frequência real | `resultados/figuras/08_calibracao.png` |
| **Curva de aprendizado** | Se mais dados ajudariam | `resultados/figuras/08_learning_curve.png` |

Plote todos os modelos na mesma figura de ROC, com a diagonal (AUC = 0,50) como
referência. Com esta base, as curvas ficam bem acima da diagonal — é a figura
mais importante do relatório.

> A **curva de aprendizado** merece atenção especial aqui: com 918 linhas, ela
> provavelmente ainda estará subindo no fim do eixo. Isso significa que mais
> dados melhorariam o modelo — uma conclusão concreta e honesta para a seção de
> próximos passos.

## 8.5 Ajuste do limiar de decisão

O corte padrão de 0,50 é convenção, não obrigação. Como o falso negativo é o
erro caro:

1. Calcule precisão e recall para limiares de 0,10 a 0,90
2. Monte a tabela limiar × precisão × recall × F1
3. Escolha o limiar que atinge o recall mínimo definido (≥ 0,85)
4. Documente a escolha e o custo em precisão

Exemplo de redação: *"ao baixar o limiar de 0,50 para 0,42, o recall subiu de
0,86 para 0,91 e a precisão caiu de 0,88 para 0,83 — trade-off aceitável, dado
que o custo de um falso negativo é maior que o de um exame adicional."*

## 8.6 Tabela comparativa final

`resultados/metricas/comparativo_modelos.csv`:

| Modelo | Acurácia | Precisão (1) | Recall (1) | F1 (1) | ROC AUC (méd ± dp) | Supera o baseline? |
|---|---|---|---|---|---|---|
| Baseline (classe majoritária) | ~0,55 | — | 1,000 | — | 0,500 | — |
| Regressão Logística | | | | | | |
| Árvore de Decisão | | | | | | |
| Random Forest | | | | | | |
| Gradient Boosting | | | | | | |

> Repare que o baseline "classe majoritária" tem recall 1,0 e AUC 0,50 — ele
> acerta todos os doentes porque chama todo mundo de doente. É exatamente por
> isso que nenhuma métrica isolada decide nada.

## 8.7 Verificação de sanidade (faça sempre)

Antes de declarar o resultado, confira:

- [ ] O alvo **não** está entre as variáveis preditoras
- [ ] Escalonamento e imputação foram ajustados **só no treino**
- [ ] A divisão foi estratificada
- [ ] `df.duplicated().sum()` é zero na base usada
- [ ] O resultado se mantém ao trocar o `random_state`
- [ ] A métrica está na faixa esperada da etapa 07

Métrica muito acima do esperado é sinal de erro, não de talento. Investigue
antes de comemorar.

## 8.8 Escolha do modelo final

Escolha considerando, nesta ordem:

1. **ROC AUC médio na validação cruzada** (com desvio baixo)
2. **Recall da classe 1** dentro da meta
3. **Interpretabilidade** — em saúde, um modelo explicável vale mais que 1 ponto
   de AUC
4. **Simplicidade** — em caso de empate técnico, o modelo mais simples vence

Justifique a escolha por escrito. Se a regressão logística ficar a 1 ponto do
Gradient Boosting, escolher a logística é defensável e demonstra maturidade.

## ✅ Checklist da etapa

- [ ] Matriz de confusão de cada modelo
- [ ] Todas as métricas calculadas para a classe 1
- [ ] Curvas ROC, PR, calibração e aprendizado geradas
- [ ] Comparação explícita contra o baseline
- [ ] Limiar de decisão avaliado e justificado
- [ ] Verificação de sanidade aprovada
- [ ] Modelo final escolhido, com justificativa escrita
- [ ] Tabela comparativa exportada

```bash
git checkout -b feature/avaliacao-modelos develop
git commit -m "feat(avaliacao): gera matriz de confusao e metricas por modelo"
git commit -m "feat(avaliacao): adiciona curvas ROC comparativas"
git commit -m "feat(avaliacao): ajusta limiar de decisao para priorizar recall"
git commit -m "docs(avaliacao): justifica escolha do modelo final"
```

➡️ Próxima: [09 — Interpretabilidade](09_interpretabilidade.md)
