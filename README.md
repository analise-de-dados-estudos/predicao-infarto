# Predição de Doença Cardíaca / Risco de Infarto — Projeto de Análise de Dados

Projeto de análise de dados e machine learning para prever a presença de doença
cardíaca a partir de variáveis clínicas reais.

O projeto segue a mesma organização usada no projeto da PRF (dados brutos →
dados tratados → análise → modelagem → resultados), com documentação passo a
passo na pasta [`docs/`](docs/) e versionamento em **Git Flow**.

---

## 1. Sobre o projeto

| Item | Descrição |
|---|---|
| **Objetivo** | Construir um modelo de classificação binária que estime a probabilidade de um paciente ter doença cardíaca |
| **Tipo de problema** | Aprendizado supervisionado — classificação binária |
| **Base principal** | [Heart Failure Prediction Dataset](https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction) — 918 registros, 12 colunas, **dados clínicos reais** |
| **Variável alvo** | `HeartDisease` (0 = sem doença, 1 = com doença) |
| **Métrica principal** | ROC AUC · métrica de negócio: recall da classe 1 |
| **Desempenho esperado** | ROC AUC entre 0,85 e 0,92 |
| **Base secundária** | Heart Attack Risk Prediction (Kaggle, sintética) — usada como contraprova |
| **Ferramentas** | Python, pandas, scikit-learn, matplotlib/seaborn, Jupyter |
| **Status** | 🟡 Em andamento |

### Pergunta de negócio

> É possível identificar, a partir de exames e sinais clínicos coletados em
> consulta, quais pacientes apresentam doença cardíaca — de forma a priorizar
> investigação diagnóstica e acompanhamento preventivo?

### Por que duas bases

A base principal (**fedesoriano**) reúne dados clínicos **reais**, consolidados
de cinco hospitais e centros de pesquisa (Cleveland, Hungria, Suíça, Long Beach
VA e Statlog). É nela que o modelo é construído e avaliado.

A base secundária (**Heart Attack Risk Prediction**, indicada em aula) é
**sintética** — gerada artificialmente. Ela entra no projeto como **experimento
de controle**: o mesmo pipeline é aplicado às duas, e a comparação mostra que a
metodologia funciona e que a diferença de resultado vem da qualidade dos dados,
não do método. O roteiro dessa comparação está em
[`docs/12_comparacao_entre_bases.md`](docs/12_comparacao_entre_bases.md).

---

## 2. Estrutura do projeto

```
predicao-infarto/
│
├── README.md                  # Este arquivo: visão geral, estrutura e padrões
├── .gitignore                 # Arquivos e pastas ignorados pelo Git
├── requirements.txt           # Bibliotecas e versões do ambiente
│
├── dados_brutos/              # 🔒 Dados originais, SEM tratamento (nunca editar)
│   ├── README.md              #    Origem, data de download e licença de cada base
│   ├── heart.csv              #    Base principal (real) - baixar do Kaggle
│   └── heart_attack_prediction_dataset.csv   # Base secundária (sintética)
│
├── dados_tratados/            # Saídas do pipeline de limpeza/transformação
│   ├── base_analitica.csv     #    Base limpa, legível, para EDA
│   ├── base_modelavel.csv     #    Base codificada/escalada, pronta para o modelo
│   └── dicionario_variaveis.csv
│
├── notebooks/                 # Jupyter Notebooks numerados na ordem de execução
│   ├── 01_exploracao_inicial.ipynb
│   ├── 02_limpeza_tratamento.ipynb
│   ├── 03_analise_exploratoria.ipynb
│   ├── 04_engenharia_atributos.ipynb
│   ├── 05_modelagem.ipynb
│   ├── 06_avaliacao_interpretacao.ipynb
│   └── 07_comparacao_bases.ipynb
│
├── sql/                       # Consultas SQL (carga, agregações, validações)
│
├── resultados/                # Tudo que é gerado pelo projeto e vai para o relatório
│   ├── figuras/               #    Gráficos exportados (.png / .svg)
│   ├── tabelas/               #    Tabelas resumo (.csv / .xlsx)
│   ├── metricas/              #    Métricas dos modelos (.csv / .json)
│   └── modelos/               #    Modelos treinados serializados (.pkl / .joblib)
│
├── logs/                      # Registros de decisões técnicas de cada etapa
│
├── referencias/               # Artigos, PDFs e materiais de apoio
│
└── docs/                      # 📘 MANUAL PASSO A PASSO DO PROJETO
    ├── 00_visao_geral.md
    ├── 01_entendimento_do_problema.md
    ├── 02_coleta_e_fontes_de_dados.md
    ├── 03_dicionario_de_variaveis.md
    ├── 04_limpeza_e_tratamento.md
    ├── 05_analise_exploratoria.md
    ├── 06_engenharia_de_atributos.md
    ├── 07_modelagem.md
    ├── 08_avaliacao_e_metricas.md
    ├── 09_interpretabilidade.md
    ├── 10_conclusao_e_entrega.md
    ├── 11_git_flow_e_padrao_de_commit.md
    └── 12_comparacao_entre_bases.md
```

> **Regra de ouro:** nada dentro de `dados_brutos/` é alterado. Todo tratamento
> gera um arquivo novo em `dados_tratados/`. Isso garante que a análise seja
> reprodutível do zero.

---

## 3. Por onde começar

1. Leia [`docs/00_visao_geral.md`](docs/00_visao_geral.md) — mapa de todas as etapas.
2. Baixe a base principal seguindo
   [`docs/02_coleta_e_fontes_de_dados.md`](docs/02_coleta_e_fontes_de_dados.md).
3. Siga os documentos de `01` a `10`, na ordem. Cada um traz **o que fazer**,
   **como decidir** e uma **checklist de conclusão**.
4. Antes do primeiro commit, leia
   [`docs/11_git_flow_e_padrao_de_commit.md`](docs/11_git_flow_e_padrao_de_commit.md).

### Preparando o ambiente

```bash
git clone https://github.com/<seu-usuario>/predicao-infarto.git
cd predicao-infarto

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
jupyter notebook
```

---

## 4. Fonte dos dados

### Base principal — dados reais

| | |
|---|---|
| **Nome** | Heart Failure Prediction Dataset |
| **Link** | https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction |
| **Autor** | fedesoriano |
| **Origem** | Consolidação de 5 bases clínicas do repositório UCI (Cleveland, Hungria, Suíça, Long Beach VA e Statlog) — 1.190 registros, 272 duplicatas removidas → **918** |
| **Volume** | 918 linhas × 12 colunas |
| **Alvo** | `HeartDisease` (0/1) |
| **Arquivo** | `dados_brutos/heart.csv` |

### Base secundária — contraprova

| | |
|---|---|
| **Nome** | Heart Attack Risk Prediction Dataset |
| **Link** | https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset |
| **Natureza** | **Sintética** (dados gerados artificialmente) |
| **Volume** | 8.763 linhas × 26 colunas |
| **Arquivo** | `dados_brutos/heart_attack_prediction_dataset.csv` (já incluído) |

Passo a passo de download (site e CLI do Kaggle) e critérios para avaliar uma
base: [`docs/02_coleta_e_fontes_de_dados.md`](docs/02_coleta_e_fontes_de_dados.md).

---

## 5. Metodologia de versionamento — Git Flow

| Branch | Papel |
|---|---|
| `main` | Versão estável e entregável. Só recebe merge de `release/*` ou `hotfix/*`. |
| `develop` | Integração do trabalho em andamento. Base das branches de feature. |
| `feature/*` | Uma etapa do projeto por branch (ex.: `feature/limpeza-dados`). |
| `release/*` | Preparação da entrega (revisão de texto, versão, ajustes finais). |
| `hotfix/*` | Correção urgente direto de `main`. |

Fluxo padrão de uma etapa:

```bash
git checkout develop
git pull origin develop
git checkout -b feature/analise-exploratoria
# ... trabalho + commits ...
git push -u origin feature/analise-exploratoria
# abrir Pull Request de feature/analise-exploratoria -> develop
```

Detalhamento completo (tags de versão, checklist de PR, comandos de emergência)
em [`docs/11_git_flow_e_padrao_de_commit.md`](docs/11_git_flow_e_padrao_de_commit.md).

---

## 6. Padrão de commit — Conventional Commits

```
<tipo>(<escopo>): <descrição no imperativo, minúscula, sem ponto final>

[corpo opcional explicando o porquê]
```

| Tipo | Quando usar |
|---|---|
| `feat` | Nova análise, novo notebook, novo modelo, nova funcionalidade |
| `fix` | Correção de erro em código, cálculo ou gráfico |
| `docs` | Alterações em README ou arquivos de `docs/` |
| `data` | Inclusão/atualização de bases em `dados_brutos/` ou `dados_tratados/` |
| `refactor` | Reorganização de código sem mudar o resultado |
| `perf` | Ganho de desempenho |
| `test` | Testes e validações |
| `chore` | Manutenção: `.gitignore`, `requirements.txt`, estrutura de pastas |
| `style` | Formatação, identação, nomes — sem efeito no resultado |

Regras:

- Descrição no **imperativo**: "adiciona", não "adicionado" ou "adicionando".
- Máximo de **72 caracteres** na primeira linha.
- **Um commit por assunto** — não misture limpeza de dados com ajuste de gráfico.
- Escopo é opcional, mas ajuda: `eda`, `limpeza`, `modelo`, `docs`, `dados`.

Exemplos reais deste projeto:

```
chore: cria estrutura inicial de pastas do projeto
data: adiciona base clinica real do Kaggle em dados_brutos
docs: adiciona manual passo a passo em docs/
feat(limpeza): trata colesterol zerado como valor ausente
feat(eda): adiciona matriz de correlacao das variaveis numericas
fix(eda): corrige eixo y invertido no grafico de faixa etaria
feat(modelo): treina regressao logistica com validacao cruzada
docs: registra comparacao entre base real e base sintetica
```

---

## 7. Entregáveis

- [ ] Bases brutas versionadas e documentadas
- [ ] Base tratada + dicionário de variáveis
- [ ] Notebook de EDA com gráficos comentados
- [ ] Modelo treinado e avaliado (com baseline de comparação)
- [ ] Comparação entre base real e base sintética
- [ ] Relatório final com conclusões e limitações
- [ ] Repositório organizado, com histórico de commits padronizado

---

## 8. Autor

**Thiago Natividade** — Curso de Análise de Dados
Projeto acadêmico, sem finalidade diagnóstica. Bases de dados sob as licenças
informadas nas respectivas páginas do Kaggle.
