# 04 — Limpeza e tratamento

> **Objetivo da etapa:** transformar `dados_brutos/heart.csv` em
> `dados_tratados/base_analitica.csv` — base limpa, com nomes padronizados e
> tipos corretos, pronta para a análise exploratória.
>
> **Notebook:** `notebooks/02_limpeza_tratamento.ipynb`

**Regra inviolável:** o arquivo de `dados_brutos/` é lido e nunca sobrescrito.
Toda saída é arquivo novo.

---

## 4.1 Ordem das operações

```
1. Carregar a base bruta e conferir shape
2. Inspecionar tipos e valores (info, describe, nunique)
3. Padronizar nomes das colunas (snake_case)
4. Identificar e tratar NULOS DISFARÇADOS   <- o ponto crítico desta base
5. Tratar valores ausentes declarados
6. Tratar duplicatas
7. Tratar inconsistências e valores impossíveis
8. Analisar outliers (analisar ≠ remover)
9. Validar a base final
10. Salvar em dados_tratados/base_analitica.csv
```

## 4.2 Diagnóstico inicial

Antes de mudar qualquer coisa, registre no notebook:

- `shape` — esperado (918, 12)
- `info()` — tipos e memória
- `isna().sum()` — nulos declarados (esperado: zero)
- `duplicated().sum()` — duplicatas (esperado: zero)
- `describe()` para numéricas e `describe(include='object')` para texto
- `value_counts()` de cada categórica

## 4.3 ⭐ O ponto crítico: colesterol zerado

Este é o achado mais importante da etapa e provavelmente o melhor parágrafo do
seu relatório.

**A situação:** cerca de **172 registros (≈19% da base)** têm `Cholesterol = 0`.
Colesterol zero é fisiologicamente impossível — uma pessoa com colesterol zero
está morta. Não é um valor, é um **exame não realizado registrado como zero**.

**Por que importa:** se você deixar como está, a média do colesterol despenca de
~240 para ~199 mg/dl, o boxplot fica deformado e todas as conclusões sobre
colesterol ficam erradas. É o tipo de erro que passa despercebido e contamina o
trabalho inteiro.

**Investigue antes de decidir:**

1. Quantos são exatamente? (`(df['colesterol'] == 0).sum()`)
2. Eles se concentram em algum subgrupo? Cruze com `doenca_cardiaca`, `sexo` e
   `tipo_dor_peito`
3. A proporção de doentes entre os zerados é diferente da do resto da base?

> Se os zeros estiverem concentrados nos pacientes com doença, o "valor ausente"
> carrega informação — provavelmente vieram de uma das cinco bases de origem
> (Suíça, por exemplo). Nesse caso, **descartar as linhas introduz viés**.

**Opções de tratamento:**

| Opção | Como | Quando escolher |
|---|---|---|
| **A — Substituir por nulo e imputar pela mediana** | `replace(0, NaN)` e depois imputar | ✅ Recomendada. Preserva as 918 linhas e não distorce a distribuição |
| **B — Imputar pela mediana do subgrupo** | Mediana por `sexo` + `doenca_cardiaca` | Mais refinada. ⚠️ Usar o alvo na imputação pode causar vazamento — se usar, impute só com estatísticas do treino |
| **C — Remover as linhas** | `drop` | ❌ Perde 19% da base e pode introduzir viés |
| **D — Criar coluna indicadora** | `colesterol_ausente` = 1 quando era zero, + imputação | ✅ Ótima. Preserva a informação "exame não realizado", que pode ser preditiva |

**Recomendação:** A + D combinadas — substitua por nulo, impute pela mediana e
crie a coluna indicadora. Depois teste na etapa 07 se a indicadora ajuda.

Faça o mesmo raciocínio com **`RestingBP = 0`** (1 registro): com uma linha só,
remover é aceitável — mas registre a decisão.

## 4.4 Valores ausentes — o raciocínio geral

| Situação | Tratamento usual |
|---|---|
| < 5% de nulos, aleatórios | Remover as linhas |
| Numérica com distribuição simétrica | Imputar pela **média** |
| Numérica assimétrica ou com outliers | Imputar pela **mediana** |
| Categórica | Imputar pela **moda** ou criar categoria `"Nao informado"` |
| Coluna com > 40% de nulos | Avaliar descartar a coluna |
| Nulo com significado | Criar coluna indicadora `flag_ausente` |

⚠️ **Nulos disfarçados** aparecem como `0`, `-1`, `999`, `"N/A"`,
`"desconhecido"` ou string vazia. Procure ativamente: `describe()` com mínimo
zero em variável que não pode ser zero é o sinal.

🚨 **Imputação e vazamento:** ao imputar com média/mediana, o valor deve ser
calculado **apenas no conjunto de treino** e aplicado ao teste. Fazer isso na
base inteira vaza informação. Na prática, use `SimpleImputer` dentro de um
`Pipeline`.

## 4.5 Duplicatas

- Duplicata **completa** (linha inteira igual): remover.
- Duplicata **parcial**: investigar antes de remover.

Nesta base, as 272 duplicatas já foram removidas pelo autor na consolidação —
por isso 1.190 viraram 918. Confirme com `df.duplicated().sum()` e registre o
resultado. **Este é o teste que separa esta base da armadilha de 1.025 linhas
citada na etapa 02.**

## 4.6 Inconsistências e valores impossíveis

| Coluna | Regra | Ação se violar |
|---|---|---|
| `idade` | 18 ≤ x ≤ 100 | Investigar |
| `pressao_repouso` | 80 ≤ x ≤ 220 | Zero → nulo disfarçado |
| `colesterol` | 100 ≤ x ≤ 700 | Zero → nulo disfarçado |
| `freq_cardiaca_max` | 60 ≤ x ≤ 220 | Investigar |
| `depressao_st` | 0 ≤ x ≤ 7 | Negativos → investigar e documentar |
| `glicemia_jejum_alta` | apenas {0, 1} | Corrigir |
| `sexo` | apenas {M, F} | Padronizar |
| `tipo_dor_peito` | apenas {TA, ATA, NAP, ASY} | Padronizar |
| `ecg_repouso` | apenas {Normal, ST, LVH} | Padronizar |
| `inclinacao_st` | apenas {Up, Flat, Down} | Padronizar |
| `doenca_cardiaca` | apenas {0, 1} | Corrigir |

Verifique também a **coerência clínica**: `freq_cardiaca_max` próxima de
`220 − idade` é o esperado; valores muito acima merecem checagem.

## 4.7 Outliers

**Analisar não é sinônimo de remover.** Em saúde, o outlier costuma ser o caso
mais importante.

Como identificar:

- **IQR:** fora de `[Q1 − 1,5·IQR ; Q3 + 1,5·IQR]`
- **Z-score:** `|z| > 3`
- **Visual:** boxplot por variável

Como decidir:

| Situação | Ação |
|---|---|
| Erro claro (colesterol = 0, pressão = 0) | Tratar como ausente |
| Valor extremo mas plausível (colesterol = 603) | **Manter** e comentar |
| Muitos outliers em uma coluna | Considerar transformação (log) em vez de remoção |

Nesta base, colesterol tem cauda longa à direita (valores acima de 400 existem e
são reais). Mantenha e comente — remover distorceria a análise.

## 4.8 Padronização de categorias

- `sexo`: `M`/`F` — decidir se mantém o código ou traduz para `Masculino`/`Feminino`
- `tipo_dor_peito`, `ecg_repouso`, `inclinacao_st`: na **base analítica**, use
  rótulos legíveis (`Assintomatico`, `Angina tipica`…). Gráfico com `ASY` no eixo
  não se explica sozinho
- A codificação numérica fica para a etapa 06

## 4.9 Validação final

- [ ] Nenhuma coluna com nome fora do padrão
- [ ] Nenhum tipo `object` que deveria ser numérico
- [ ] Nenhum zero disfarçado remanescente
- [ ] Nenhuma regra da tabela 4.6 violada
- [ ] Linhas: partiu de 918, terminou com ____ (justifique a diferença)
- [ ] Alvo com duas categorias e proporção próxima de 55/45
- [ ] `describe()` sem valores absurdos

## 4.10 Saída da etapa

| Arquivo | Conteúdo |
|---|---|
| `dados_tratados/base_analitica.csv` | Base limpa, com rótulos legíveis — usada na EDA |
| `dados_tratados/dicionario_variaveis.csv` | Atualizado com as colunas derivadas |
| `logs/04_limpeza.md` | Registro das decisões: o que mudou, quantas linhas, por quê |

> A base **modelável** (tudo numérico, codificado e escalado) só nasce na
> etapa 06. Não misture as duas: gráfico com `1` e `0` no eixo é ilegível.

## ✅ Checklist da etapa

- [ ] Base bruta lida sem alteração do arquivo original
- [ ] Diagnóstico inicial registrado no notebook
- [ ] Nomes padronizados
- [ ] **Colesterol zerado investigado, tratado e a decisão justificada**
- [ ] `RestingBP = 0` tratado
- [ ] `Oldpeak` negativo investigado
- [ ] Duplicatas verificadas (esperado: zero)
- [ ] Outliers analisados com decisão justificada
- [ ] Validação final aprovada
- [ ] `base_analitica.csv` salva
- [ ] Decisões registradas em `logs/04_limpeza.md`

```bash
git checkout -b feature/limpeza-dados develop
git commit -m "feat(limpeza): padroniza nomes das colunas em snake_case"
git commit -m "feat(limpeza): trata colesterol zerado como valor ausente"
git commit -m "feat(limpeza): cria indicadora de colesterol nao informado"
git commit -m "data: gera base analitica tratada a partir da base bruta"
```

➡️ Próxima: [05 — Análise exploratória](05_analise_exploratoria.md)
