# 06 — Engenharia de atributos

> **Objetivo da etapa:** transformar a base analítica em uma base 100% numérica,
> codificada e escalada, pronta para os algoritmos.
>
> **Notebook:** `notebooks/04_engenharia_atributos.ipynb`
> **Entrada:** `dados_tratados/base_analitica.csv`
> **Saída:** `dados_tratados/base_modelavel.csv`

---

## 6.1 Criação de variáveis derivadas

Variáveis novas com significado clínico. Cada uma precisa de justificativa —
e cada uma precisa ser testada, porque nem toda derivada ajuda.

| Nova variável | Como é construída | Por quê |
|---|---|---|
| `colesterol_ausente` | 1 se o valor original era 0 | Preserva a informação "exame não realizado", que pode ser preditiva |
| `faixa_etaria` | 28–39, 40–49, 50–59, 60–69, 70+ | Captura efeito não linear da idade |
| `fc_max_esperada` | `220 − idade` | Referência fisiológica de frequência máxima |
| `reserva_cardiaca` | `freq_cardiaca_max − fc_max_esperada` | Quanto o paciente ficou abaixo do esperado para a idade — indicador de limitação funcional |
| `colesterol_alto` | 1 se `colesterol` > 240 | Corte clínico usual |
| `pressao_elevada` | 1 se `pressao_repouso` ≥ 140 | Critério de hipertensão |
| `st_alterado` | 1 se `inclinacao_st` ≠ `Up` | Agrupa `Flat` e `Down`, que são os padrões suspeitos |
| `escore_esforco` | Combinação de `angina_exercicio`, `depressao_st` e `st_alterado` | Agrega os sinais do teste ergométrico em um indicador |

> `reserva_cardiaca` costuma ser a derivada mais útil desta base — teste-a. Use
> as faixas categóricas na EDA e no relatório (são legíveis) e teste as duas
> versões no modelo: árvores costumam ir melhor com o valor contínuo.

## 6.2 Codificação de variáveis categóricas (encoding)

| Variável | Técnica | Resultado |
|---|---|---|
| `sexo` | Binária | `M` → 1, `F` → 0 |
| `angina_exercicio` | Binária | `Y` → 1, `N` → 0 |
| `tipo_dor_peito` | **One-hot** | 4 categorias → 3 colunas (`drop_first`). Não há ordem natural |
| `ecg_repouso` | **One-hot** | 3 categorias → 2 colunas |
| `inclinacao_st` | **Ordinal** ou one-hot | Há ordem de risco (`Up` → `Flat` → `Down`). **Teste as duas** e compare |
| `faixa_etaria` | Ordinal | Mantém a ordem |

⚠️ Nunca aplique *label encoding* (0, 1, 2, 3…) em categoria **sem ordem** — o
modelo passa a achar que `ASY > NAP > ATA`. Este é o erro de encoding mais comum
em projeto de curso.

> A `inclinacao_st` é a variável mais importante desta base. Vale mesmo testar
> ordinal versus one-hot e reportar qual funcionou melhor — é um teste barato
> que enriquece o relatório.

## 6.3 Colunas a descartar

Esta base tem poucas colunas, então descarte com parcimônia:

| Coluna | Motivo possível |
|---|---|
| `fc_max_esperada` | É função direta da idade — redundante se `reserva_cardiaca` já existe |
| `colesterol_alto` | Redundante com `colesterol` contínuo, se não ajudar |
| Derivadas que não melhoram a métrica | Higiene |

Não existe coluna de identificação para descartar. Registre cada descarte com o
motivo.

## 6.4 Escalonamento (normalização/padronização)

| Técnica | Quando usar | Fórmula |
|---|---|---|
| `StandardScaler` | Padrão para regressão logística, SVM, KNN | (x − média) / desvio |
| `MinMaxScaler` | Quando é necessário faixa fixa [0, 1] | (x − mín) / (máx − mín) |
| `RobustScaler` | Quando há muitos outliers — útil para `colesterol` | usa mediana e IQR |
| **Nenhum** | Árvores de decisão, Random Forest, XGBoost | são insensíveis à escala |

🚨 **Erro clássico:** ajustar o scaler na base inteira. O correto é `fit`
**só no treino** e `transform` no teste — caso contrário há vazamento de
informação e as métricas ficam otimistas. Na prática, use `Pipeline` +
`ColumnTransformer`.

Nesta base, `colesterol` (100–600) contra `depressao_st` (0–6) torna o
escalonamento obrigatório para modelos baseados em distância.

## 6.5 Seleção de atributos

Com apenas 11 preditores, a seleção agressiva não costuma valer a pena — o risco
de perder informação é maior que o ganho. Mesmo assim, execute para documentar:

1. **Correlação alta entre preditoras** — se |r| > 0,80, manter uma
2. **Seleção univariada** — `SelectKBest` com ANOVA (numéricas) ou qui-quadrado
   (categóricas)
3. **Importância por modelo** — Random Forest ou coeficientes da regressão
4. **RFE (eliminação recursiva)** — barato com 11 variáveis

Compare a métrica com todas as variáveis versus com o subconjunto selecionado.
Se não melhorar, mantenha todas e registre a decisão.

## 6.6 Balanceamento de classes

A base está em ~55% / ~45% — praticamente balanceada. **Não use SMOTE aqui.**
Criar exemplos sintéticos numa base já equilibrada só adiciona ruído.

Se quiser priorizar recall mesmo assim:

| Técnica | Como funciona | Observação |
|---|---|---|
| `class_weight='balanced'` | Penaliza mais o erro na classe minoritária | Efeito pequeno nesta base |
| **Ajuste de limiar** | Baixar o corte de 0,50 para ~0,40 | ✅ Caminho mais simples e eficaz aqui |

🚨 Se em algum momento usar SMOTE (na base sintética da etapa 12, por exemplo),
aplique **somente ao treino**, depois da divisão. Aplicar antes é o segundo erro
mais comum do curso.

## 6.7 Divisão treino / teste

- Proporção: **80/20**
- **Estratificada** pelo alvo (`stratify=y`)
- `random_state` fixo (ex.: 42) — sem isso, o resultado não é reproduzível
- **Validação cruzada é obrigatória nesta base.** Com 918 linhas, o conjunto de
  teste tem ~184 registros: a métrica varia bastante conforme o sorteio. Use
  `StratifiedKFold` com 5 ou 10 folds e reporte média ± desvio

**A divisão acontece ANTES de escalonar, imputar ou balancear.** Sempre.

## 6.8 Saída da etapa

| Arquivo | Conteúdo |
|---|---|
| `dados_tratados/base_modelavel.csv` | Tudo numérico, codificado, sem colunas descartadas |
| `dados_tratados/dicionario_variaveis.csv` | Atualizado com as derivadas (`origem = derivada`) |
| `logs/06_features.md` | Variáveis criadas, mantidas e descartadas, com justificativa |

## ✅ Checklist da etapa

- [ ] Variáveis derivadas criadas e justificadas
- [ ] Categóricas codificadas com a técnica correta (ordinal × one-hot)
- [ ] `inclinacao_st` testada nas duas codificações
- [ ] Colunas descartadas com motivo registrado
- [ ] Escalonamento definido conforme o modelo
- [ ] SMOTE **não** aplicado (base balanceada)
- [ ] Divisão treino/teste estratificada, com `random_state` fixo
- [ ] Escalonamento e imputação aplicados **apenas ao treino** (via Pipeline)
- [ ] `base_modelavel.csv` salva
- [ ] Dicionário atualizado

```bash
git checkout -b feature/engenharia-atributos develop
git commit -m "feat(features): cria reserva cardiaca e faixas etarias derivadas"
git commit -m "feat(features): aplica one-hot em tipo de dor e ordinal em inclinacao st"
git commit -m "data: gera base modelavel pronta para treinamento"
```

➡️ Próxima: [07 — Modelagem](07_modelagem.md)
