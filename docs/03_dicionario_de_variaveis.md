# 03 — Dicionário de variáveis

> **Objetivo da etapa:** documentar cada coluna da base — tipo, significado,
> domínio de valores e decisão de uso. Sem isso, você não sabe o que está
> modelando.

---

## 3.1 Base principal — `heart.csv` (918 × 12)

| # | Coluna | Tipo | Domínio | Significado clínico | Uso |
|---|---|---|---|---|---|
| 1 | `Age` | inteiro | ~28 – 77 | Idade em anos | ✅ Numérica |
| 2 | `Sex` | texto | `M`, `F` | Sexo biológico | ✅ Binária (encoding) |
| 3 | `ChestPainType` | texto | `TA`, `ATA`, `NAP`, `ASY` | Tipo de dor no peito | ✅ One-hot |
| 4 | `RestingBP` | inteiro | 0 – ~200 mm Hg | Pressão arterial em repouso | ⚠️ 1 registro com 0 |
| 5 | `Cholesterol` | inteiro | 0 – ~603 mg/dl | Colesterol sérico | ⚠️ **~172 registros com 0** |
| 6 | `FastingBS` | binária | 0 / 1 | 1 se glicemia em jejum > 120 mg/dl | ✅ Direto |
| 7 | `RestingECG` | texto | `Normal`, `ST`, `LVH` | Resultado do ECG em repouso | ✅ One-hot |
| 8 | `MaxHR` | inteiro | ~60 – 202 | Frequência cardíaca máxima atingida no esforço | ✅ Numérica |
| 9 | `ExerciseAngina` | texto | `Y`, `N` | Angina induzida por exercício | ✅ Binária |
| 10 | `Oldpeak` | decimal | ~−2,6 – 6,2 | Depressão do segmento ST induzida por exercício | ✅ Numérica |
| 11 | `ST_Slope` | texto | `Up`, `Flat`, `Down` | Inclinação do segmento ST no pico do exercício | ✅ One-hot ou ordinal |
| 12 | `HeartDisease` | binária | 0 / 1 (~45% / ~55%) | 🎯 **Alvo** — doença arterial coronariana confirmada | 🎯 Alvo |

> Os intervalos acima são referência. **Confirme todos no seu arquivo** com
> `df.describe()` e registre os valores reais — a base pode ter sido atualizada.

### Legenda das categorias

**`ChestPainType` — tipo de dor no peito**

| Código | Significado | Observação |
|---|---|---|
| `TA` | *Typical Angina* — angina típica | Dor clássica de origem cardíaca |
| `ATA` | *Atypical Angina* — angina atípica | Dor com características parciais |
| `NAP` | *Non-Anginal Pain* — dor não anginosa | Origem provavelmente não cardíaca |
| `ASY` | *Asymptomatic* — assintomático | ⚠️ Maior proporção de doença na base |

**`RestingECG` — eletrocardiograma em repouso**

| Código | Significado |
|---|---|
| `Normal` | Sem alterações |
| `ST` | Anormalidade da onda T ou do segmento ST |
| `LVH` | Hipertrofia ventricular esquerda (critério de Estes) |

**`ST_Slope` — inclinação do segmento ST no pico do exercício**

| Código | Significado | Associação típica |
|---|---|---|
| `Up` | Ascendente | Padrão normal — menor probabilidade de doença |
| `Flat` | Plana | Suspeito |
| `Down` | Descendente | Mais associado a doença |

Existe **ordem natural** (`Up` → `Flat` → `Down` = risco crescente), então esta
variável admite encoding ordinal. Teste as duas formas na etapa 06.

**`Oldpeak`** — depressão do segmento ST em milímetros, medida durante o teste
de esforço em relação ao repouso. Valores maiores indicam maior isquemia.
Valores negativos são fisiologicamente atípicos: investigue e documente a
decisão.

## 3.2 Observações que impactam as próximas etapas

| Achado | Consequência |
|---|---|
| `Cholesterol = 0` em ~172 registros (≈19% da base) | **Nulo disfarçado.** É a decisão mais importante da etapa 04 |
| `RestingBP = 0` em 1 registro | Nulo disfarçado — remover a linha ou imputar |
| `Oldpeak` com valores negativos | Verificar quantos e decidir tratamento |
| Alvo em ~55% / ~45% | Base quase balanceada — não exige SMOTE |
| Apenas 918 linhas | Validação cruzada é obrigatória; uma única divisão treino/teste é instável |
| `Sex` fortemente desbalanceado (mais homens) | Cuidado ao generalizar conclusões para mulheres |
| Nenhuma coluna de identificação | Nada a descartar por esse motivo |

## 3.3 Padronização de nomes

Nomes em inglês e em CamelCase atrapalham. Padrão para a base tratada:
**snake_case, minúsculas, sem acento**.

| Original | Padronizado |
|---|---|
| `Age` | `idade` |
| `Sex` | `sexo` |
| `ChestPainType` | `tipo_dor_peito` |
| `RestingBP` | `pressao_repouso` |
| `Cholesterol` | `colesterol` |
| `FastingBS` | `glicemia_jejum_alta` |
| `RestingECG` | `ecg_repouso` |
| `MaxHR` | `freq_cardiaca_max` |
| `ExerciseAngina` | `angina_exercicio` |
| `Oldpeak` | `depressao_st` |
| `ST_Slope` | `inclinacao_st` |
| `HeartDisease` | `doenca_cardiaca` |

> Escolha um idioma e mantenha. Traduzir metade das colunas é pior que não
> traduzir nenhuma. Se preferir manter os nomes originais em inglês, tudo bem —
> só seja consistente e documente a escolha.

## 3.4 Arquivo de saída

Gere `dados_tratados/dicionario_variaveis.csv` — mesmo formato usado no projeto
da PRF, o que facilita a comparação entre os dois trabalhos:

```
variavel;tipo;nulos_%;exemplo;origem;descricao
```

Onde `origem` é `original` (veio da base bruta) ou `derivada` (criada por você
nas etapas 04/06). Para esta base, `nulos_%` deve considerar os **zeros
disfarçados** — colesterol zerado conta como ausente, não como zero.

## 3.5 Apêndice — base secundária (sintética, 26 colunas)

Necessário apenas para a etapa 12.

| Grupo | Colunas |
|---|---|
| Identificação | `Patient ID` (descartar) |
| Demográficas | `Age`, `Sex`, `Country`, `Continent`, `Hemisphere` |
| Clínicas | `Cholesterol`, `Blood Pressure` (texto `"158/88"` — separar), `Heart Rate`, `BMI`, `Triglycerides` |
| Condições | `Diabetes`, `Family History`, `Obesity`, `Previous Heart Problems`, `Medication Use` |
| Estilo de vida | `Smoking`, `Alcohol Consumption`, `Diet`, `Exercise Hours Per Week`, `Sedentary Hours Per Day`, `Physical Activity Days Per Week`, `Sleep Hours Per Day`, `Stress Level` |
| Socioeconômica | `Income` |
| Alvo | `Heart Attack Risk` (64% / 36%) |

Pontos verificados: zero nulos, zero duplicatas, distribuições contínuas
uniformes e correlação máxima com o alvo de **0,019**.

## ✅ Checklist da etapa

- [ ] Todas as 12 colunas da base principal descritas
- [ ] Códigos das categóricas traduzidos (`ASY`, `ATA`, `LVH`, `ST_Slope`…)
- [ ] Tipos e domínios conferidos **no arquivo**, não copiados deste manual
- [ ] Zeros disfarçados quantificados
- [ ] Distribuição do alvo registrada
- [ ] Padrão de nomes definido
- [ ] `dicionario_variaveis.csv` gerado em `dados_tratados/`

```bash
git checkout -b feature/dicionario-dados develop
git add docs/03_dicionario_de_variaveis.md dados_tratados/dicionario_variaveis.csv
git commit -m "docs: adiciona dicionario das 12 variaveis da base clinica"
```

➡️ Próxima: [04 — Limpeza e tratamento](04_limpeza_e_tratamento.md)
