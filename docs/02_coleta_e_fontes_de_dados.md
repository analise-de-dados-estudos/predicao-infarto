# 02 — Coleta e fontes de dados

> **Objetivo da etapa:** ter as bases brutas dentro de `dados_brutos/`, com
> origem, data de download e licença documentadas.

---

## 2.1 Base principal — Heart Failure Prediction (dados reais)

| | |
|---|---|
| **Link** | https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction |
| **Autor** | fedesoriano |
| **Volume** | 918 linhas × 12 colunas |
| **Alvo** | `HeartDisease` (0 = sem doença, 1 = com doença) |
| **Natureza** | **Real** — dados clínicos coletados em hospitais |
| **Arquivo** | `dados_brutos/heart.csv` |

### Por que esta base

1. **Dados reais e rastreáveis.** É a consolidação de cinco bases do repositório
   UCI, referência acadêmica no tema desde 1988:

   | Base de origem | Registros |
   |---|---|
   | Cleveland | 303 |
   | Hungria | 294 |
   | Suíça | 123 |
   | Long Beach VA | 200 |
   | Statlog (Heart) | 270 |
   | **Total bruto** | **1.190** |
   | Duplicatas removidas | 272 |
   | **Total final** | **918** |

2. **Variáveis com significado clínico.** São os preditores que a cardiologia
   realmente usa — resultado de teste ergométrico, tipo de dor no peito, ECG em
   repouso —, não indicadores genéricos de estilo de vida.

3. **O modelo funciona.** ROC AUC entre 0,85 e 0,92 com modelos simples. Você
   terá curva ROC bem acima da diagonal, importância de variáveis estável e
   coerente com a literatura.

4. **Tem sujeira de verdade.** Os valores zerados de colesterol dão conteúdo
   real para a etapa de limpeza.

5. **Tamanho adequado ao prazo.** 918 linhas rodam em segundos; o tempo vai para
   a análise, não para a espera.

### Como baixar pelo site

1. Crie uma conta gratuita em https://www.kaggle.com.
2. Abra o link da base acima.
3. Aba **Data** → botão **Download** → baixa um `.zip` com `heart.csv`.
4. Descompacte e mova `heart.csv` para `dados_brutos/`.
5. Anote a **data do download** e a **licença** em `dados_brutos/README.md`.

### Como baixar pela linha de comando (reprodutível)

```bash
pip install kaggle

# 1. No Kaggle: Settings > API > "Create New Token" -> baixa kaggle.json
# 2. Coloque o arquivo em:
#    Linux/Mac -> ~/.kaggle/kaggle.json   (depois: chmod 600 ~/.kaggle/kaggle.json)
#    Windows   -> C:\Users\<usuario>\.kaggle\kaggle.json

kaggle datasets download -d fedesoriano/heart-failure-prediction -p dados_brutos/
unzip dados_brutos/heart-failure-prediction.zip -d dados_brutos/
```

> ⚠️ **Nunca** faça commit do `kaggle.json` — é sua credencial. Já está bloqueado
> no `.gitignore` deste projeto.

### Verificação obrigatória ao abrir

Rode e registre no notebook `01_exploracao_inicial.ipynb`:

| Verificação | Esperado |
|---|---|
| `df.shape` | (918, 12) |
| `df.duplicated().sum()` | 0 |
| `df.isna().sum()` | 0 nulos declarados |
| `(df['Cholesterol'] == 0).sum()` | ~172 → **nulos disfarçados** |
| `(df['RestingBP'] == 0).sum()` | 1 → nulo disfarçado |
| `df['Oldpeak'].min()` | valor negativo → investigar |
| `df['HeartDisease'].value_counts(normalize=True)` | ~55% classe 1 / ~45% classe 0 |

Se algum número divergir, registre o valor real — a base pode ter sido
atualizada. **Nunca copie um número deste manual para o relatório sem conferir
no seu arquivo.**

---

## 2.2 Base secundária — contraprova (sintética)

| | |
|---|---|
| **Link** | https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset |
| **Volume** | 8.763 linhas × 26 colunas |
| **Natureza** | **Sintética** — gerada artificialmente |
| **Arquivo** | `dados_brutos/heart_attack_prediction_dataset.csv` (já no repositório) |

É a base indicada em aula. Ela permanece no projeto com um papel definido:
**experimento de controle**. Na etapa 12, o mesmo pipeline é aplicado às duas
bases e as métricas são comparadas lado a lado.

Verificações já feitas neste arquivo:

- 8.763 linhas, 26 colunas, **zero nulos**, **zero duplicatas**
- `Blood Pressure` vem como texto (`"158/88"`) — precisaria ser separada
- Distribuições contínuas **uniformes** (assinatura de valor sorteado)
- Maior correlação absoluta com o alvo: **0,019** (colesterol)

---

## 2.3 Armadilha a evitar

⚠️ **[Heart Disease Dataset — johnsmith88](https://www.kaggle.com/datasets/johnsmith88/heart-disease-dataset) (1.025 linhas).**

É a base mais popular do tema, e você encontrará dezenas de notebooks
anunciando 98%, 99% ou 100% de acurácia. Ela é a Cleveland original (303
registros) com linhas **duplicadas**. Na divisão treino/teste, a mesma linha cai
dos dois lados: o modelo reconhece um registro já visto em vez de generalizar.
É vazamento de dados, e a métrica é falsa.

**Regra:** rode `df.duplicated().sum()` em qualquer base antes de confiar em
qualquer métrica. Acurácia acima de 95% em problema clínico é motivo de
desconfiança, não de comemoração.

---

## 2.4 Outras bases reais (se quiser ampliar o trabalho)

| Base | Volume | Quando usar |
|---|---|---|
| [Heart Attack Analysis & Prediction](https://www.kaggle.com/datasets/rashikrahmanpritom/heart-attack-analysis-prediction-dataset) | 303 × 14 | A Cleveland pura, clássica dos artigos acadêmicos |
| [Heart Disease Health Indicators](https://www.kaggle.com/datasets/alexteboul/heart-disease-health-indicators-dataset) | ~253.680 × 22 | Pesquisa real do CDC (BRFSS 2015). Volume grande e desbalanceamento forte (~9% positivos) |
| [Personal Key Indicators of Heart Disease](https://www.kaggle.com/datasets/kamilpytlak/personal-key-indicators-of-heart-disease) | ~320 mil linhas | Mesma origem BRFSS, com variáveis de estilo de vida |
| [Heart Disease — UCI (fonte original)](https://archive.ics.uci.edu/dataset/45/heart+disease) | 303 × 14 | Citar a fonte primária no relatório (licença CC BY 4.0) |

### Contexto brasileiro para a introdução

| Fonte | O que tem | Link |
|---|---|---|
| **DATASUS / TabNet** | Internações e óbitos por infarto agudo do miocárdio (SIH/SIM), por município, idade e sexo | https://datasus.saude.gov.br/informacoes-de-saude-tabnet/ |
| **IBGE — PNS** | Pesquisa Nacional de Saúde, com dados individuais | https://www.ibge.gov.br/estatisticas/sociais/saude |

São dados **agregados** — não servem para classificação paciente a paciente, mas
dão ótimo contexto na introdução ("no Brasil, X internações por IAM em 2025").

---

## 2.5 Como avaliar se uma base presta

- [ ] **Origem declarada?** Diz de onde vieram os dados ou é "gerada"?
- [ ] **Duplicatas?** `df.duplicated().sum()` — o teste que revela vazamento
- [ ] **Usability score** ≥ 8,0 no Kaggle
- [ ] **Licença** compatível com uso acadêmico
- [ ] **Dicionário de variáveis** disponível na página
- [ ] **Alvo bem definido** e sem vazamento (nenhuma coluna é consequência do alvo)
- [ ] **Correlações plausíveis** — se nenhuma variável se correlaciona com o
      alvo, provavelmente é dado aleatório
- [ ] **Notebooks públicos:** se ninguém passa de ~0,55 de AUC, o problema é a
      base; se todo mundo passa de 0,98, provavelmente há duplicatas

## ✅ Checklist da etapa

- [ ] `heart.csv` baixada e colocada em `dados_brutos/` **sem alteração**
- [ ] `dados_brutos/README.md` preenchido (URL, data, licença, volume)
- [ ] Verificações da seção 2.1 executadas e os números registrados
- [ ] Base secundária mantida e documentada
- [ ] Credenciais (`kaggle.json`) fora do repositório
- [ ] Commit realizado

```bash
git checkout -b feature/coleta-dados develop
git add dados_brutos/
git commit -m "data: adiciona base clinica real do Kaggle e documenta a origem"
```

➡️ Próxima: [03 — Dicionário de variáveis](03_dicionario_de_variaveis.md)
