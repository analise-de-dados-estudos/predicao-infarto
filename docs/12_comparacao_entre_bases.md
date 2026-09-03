# 12 — Comparação entre bases: real × sintética

> **Objetivo da etapa:** aplicar o mesmo pipeline às duas bases e demonstrar,
> com números, que a diferença de resultado vem da **qualidade dos dados**, não
> do método.
>
> **Notebook:** `notebooks/07_comparacao_bases.ipynb`
> **Execute depois da etapa 09 e antes da 10.**

---

## 12.1 Por que esta etapa existe

O professor indicou a base sintética. Você trocou para uma base real — e precisa
justificar a troca com evidência, não com opinião. Esta etapa faz exatamente
isso, e de quebra vira o diferencial do trabalho.

O raciocínio é o de um **experimento controlado**: mesmo pipeline, mesmos
modelos, mesma métrica, mesma validação. A única coisa que muda é a base. Se os
resultados divergirem radicalmente, a causa está isolada.

## 12.2 O que manter idêntico

Para a comparação ser válida, nada além da base pode mudar:

| Elemento | Configuração |
|---|---|
| Modelos | Os mesmos 4 ou 5 da etapa 07 |
| Divisão | 80/20 estratificada, `random_state=42` |
| Validação | `StratifiedKFold(5)` |
| Métrica | ROC AUC (média ± desvio) |
| Pré-processamento | Mesmo tipo de encoding e escalonamento |
| Baseline | `DummyClassifier` em ambas |

Adapte apenas o que é obrigatório: a base sintética exige separar
`Blood Pressure` (texto `"158/88"`) e descartar `Patient ID`.

## 12.3 Tabela principal do relatório

| | Base real (fedesoriano) | Base sintética (Kaggle) |
|---|---|---|
| Registros | 918 | 8.763 |
| Variáveis | 12 | 26 |
| Origem | 5 hospitais/centros (UCI) | Gerada artificialmente |
| Nulos | 0 declarados, ~172 disfarçados | 0 |
| Duplicatas | 0 | 0 |
| Distribuição do alvo | ~55% / ~45% | 64% / 36% |
| **Maior \|correlação\| com o alvo** | *preencher* | **0,019** |
| Distribuições contínuas | Normais / assimétricas | **Uniformes** |
| Baseline (ROC AUC) | 0,50 | 0,50 |
| Regressão Logística | *preencher* | ~0,50 |
| Random Forest | *preencher* | ~0,50 |
| Gradient Boosting | *preencher* | ~0,50 |
| **Ganho sobre o baseline** | *preencher* | **≈ zero** |
| Estabilidade do ranking de variáveis | Estável | Muda a cada semente |

Preencha com os **seus** números. Os da coluna da direita já foram verificados
neste projeto; os da esquerda saem das etapas 07 e 08.

## 12.4 As três figuras que provam o argumento

Estas três valem mais que qualquer parágrafo:

1. **Curvas ROC lado a lado.** Um painel com dois gráficos: à esquerda, as
   curvas da base real bem acima da diagonal; à direita, as curvas da base
   sintética coladas na diagonal. É a imagem que resume o trabalho inteiro.

2. **Histogramas comparados.** Idade e colesterol nas duas bases, mesma escala.
   A real tem forma de sino; a sintética é um retângulo. Distribuição uniforme
   em variável biológica não existe na natureza — é assinatura de valor
   sorteado.

3. **Heatmaps de correlação lado a lado.** Na real, um padrão visível de
   associações; na sintética, um bloco uniformemente sem cor.

## 12.5 Estrutura da seção no relatório

```
1. Objetivo da comparação (1 parágrafo)
2. Metodologia: o que foi mantido constante (1 parágrafo + tabela 12.2)
3. Resultados: tabela 12.3 + as três figuras
4. Discussão: por que a base sintética não permite predição
5. Conclusão metodológica
```

### Redação sugerida para a conclusão

> "O mesmo pipeline de pré-processamento, modelagem e validação foi aplicado às
> duas bases. Na base clínica real, os modelos alcançaram ROC AUC de [X], contra
> 0,50 do baseline trivial. Na base sintética, nenhum modelo superou o baseline
> de forma consistente (ROC AUC entre [Y] e [Z]).
>
> A investigação identificou a causa: na base sintética, a maior correlação
> absoluta entre qualquer variável preditora e o alvo é de 0,019, e as
> distribuições das variáveis contínuas são uniformes — comportamento
> incompatível com dados biológicos e consistente com geração aleatória,
> conforme declarado pelo próprio autor da base.
>
> Como o método foi idêntico nos dois casos, a diferença de desempenho é
> atribuível exclusivamente à natureza dos dados. Isso valida o pipeline
> construído e demonstra que nenhum algoritmo compensa a ausência de sinal na
> base de origem — avaliar criticamente a qualidade dos dados é parte
> indispensável do trabalho do analista, e não uma etapa opcional."

## 12.6 Como apresentar isso ao professor

Não é "não usei a base que você indicou". É:

> "Usei a base indicada como **experimento de controle**. Ao analisá-la,
> identifiquei que se trata de dados sintéticos sem correlação com o alvo, o que
> impossibilita qualquer predição. Para demonstrar isso de forma rigorosa, apliquei
> o mesmo pipeline a uma base clínica real e comparei os resultados."

Você não descartou a indicação — você a investigou e ampliou o escopo. É a
diferença entre executar uma tarefa e conduzir uma análise.

## ✅ Checklist da etapa

- [ ] Pipeline aplicado às duas bases sem alterações metodológicas
- [ ] Mesmos modelos, mesma divisão, mesmo `random_state`
- [ ] Tabela comparativa preenchida com números próprios
- [ ] Painel de curvas ROC lado a lado gerado
- [ ] Histogramas comparados gerados
- [ ] Heatmaps de correlação lado a lado gerados
- [ ] Seção do relatório escrita
- [ ] Conclusão metodológica redigida

```bash
git checkout -b feature/comparacao-bases develop
git commit -m "feat(comparacao): aplica pipeline a base sintetica para contraprova"
git commit -m "feat(comparacao): gera figuras comparativas de ROC e distribuicoes"
git commit -m "docs(comparacao): registra conclusao metodologica da comparacao"
```

➡️ Próxima: [10 — Conclusão e entrega](10_conclusao_e_entrega.md)
