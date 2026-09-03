# 01 — Entendimento do problema

> **Objetivo da etapa:** sair daqui com a pergunta de negócio escrita, as
> hipóteses listadas e a métrica de sucesso escolhida — antes de abrir qualquer
> planilha.

Esta é a etapa que mais se pula e a que mais custa caro depois. Sem ela, você
gera trinta gráficos bonitos que não respondem nada.

---

## 1.1 Pergunta de negócio

Uma frase, respondível com dados, com sujeito e objetivo claros.

**Deste projeto:**

> É possível identificar, a partir de exames e sinais clínicos coletados em
> consulta, quais pacientes apresentam doença cardíaca — de forma a priorizar
> investigação diagnóstica e acompanhamento preventivo?

Perguntas de apoio (viram seções da EDA):

1. O perfil de risco muda com a idade e o sexo?
2. O tipo de dor no peito discrimina os pacientes com e sem doença?
3. Indicadores de esforço (frequência cardíaca máxima, angina induzida por
   exercício, depressão do segmento ST) separam os grupos?
4. Colesterol e pressão em repouso, isoladamente, têm poder discriminante?
5. Qual é a variável isolada mais associada ao diagnóstico?

## 1.2 Contexto clínico mínimo

Você não precisa ser médico, mas precisa entender o que está modelando. Em
resumo:

- O alvo `HeartDisease` indica **doença arterial coronariana** confirmada por
  angiografia (estreitamento maior que 50% em ao menos uma artéria principal).
  É o quadro que antecede o infarto.
- As variáveis mais fortes vêm do **teste ergométrico** (esteira): `ST_Slope`,
  `Oldpeak`, `ExerciseAngina` e `MaxHR`. Isso faz sentido clínico — é sob
  esforço que a irrigação insuficiente do coração se manifesta.
- `ChestPainType = ASY` (assintomático) é, contraintuitivamente, o grupo com
  **maior** proporção de doença nesta base. Explicação provável: são pacientes
  encaminhados à angiografia por outros indícios, não por dor. Vale um
  parágrafo no relatório.

## 1.3 Tipo de problema

| Pergunta | Resposta deste projeto |
|---|---|
| Supervisionado ou não supervisionado? | **Supervisionado** — existe rótulo (`HeartDisease`) |
| Classificação ou regressão? | **Classificação binária** (0 / 1) |
| O que o modelo entrega? | Uma **probabilidade** de doença, que vira classe ao aplicar um limiar |
| Erro mais caro | **Falso negativo** — dizer "sem doença" a quem tem |

Essa última linha define tudo na etapa 08: como o falso negativo é o erro mais
grave, o modelo será avaliado priorizando **recall** (sensibilidade) da classe 1,
e não a acurácia.

## 1.4 Hipóteses a testar

Escreva **antes** de olhar os dados — assim você não confunde descoberta com
confirmação do que já esperava.

| # | Hipótese | Como testar (etapa 05) |
|---|---|---|
| H1 | Pacientes mais velhos têm maior proporção de doença | Taxa de doença por faixa etária |
| H2 | Homens apresentam maior proporção de doença | Tabela cruzada `Sex` × alvo |
| H3 | Angina induzida por exercício se associa fortemente ao diagnóstico | Tabela cruzada `ExerciseAngina` × alvo + qui-quadrado |
| H4 | Frequência cardíaca máxima é **menor** no grupo com doença | Boxplot `MaxHR` por classe + teste t/Mann-Whitney |
| H5 | `ST_Slope` plana ou descendente indica doença | Barras empilhadas 100% |
| H6 | Colesterol isolado é um preditor fraco | Boxplot + correlação com o alvo |

Cada hipótese termina com um veredito: **confirmada**, **refutada** ou
**inconclusiva** — e todas as três são resultados válidos.

> H6 é proposital: colesterol alto é o fator que o senso comum associa a infarto,
> mas nesta base ele discrimina pouco (e ainda tem o problema dos zeros). Um bom
> parágrafo sobre isso mostra leitura crítica.

## 1.5 Métrica de sucesso

| Métrica | Papel neste projeto |
|---|---|
| **ROC AUC** | Métrica principal — mede separação entre as classes independentemente do limiar |
| **Recall (classe 1)** | Métrica de negócio — quantos pacientes doentes o modelo captura |
| **Precisão (classe 1)** | Controle — quanto do alerta é ruído |
| **F1-score** | Equilíbrio entre as duas acima |
| **Acurácia** | Informativa. Como a base é quase balanceada (~55% / 45%), aqui ela engana menos que o normal — mas ainda não é a métrica principal |

**Meta:** ROC AUC ≥ 0,85 com recall da classe 1 ≥ 0,85.

**Baseline obrigatório:** antes de qualquer modelo sofisticado, registre o
desempenho do modelo trivial (prever sempre a classe majoritária) e de uma
regressão logística simples. Modelo novo só entra no relatório se superar o
baseline.

## 1.6 Escopo e limitações (declarar desde já)

- Projeto **acadêmico**. Não é ferramenta de diagnóstico e não substitui
  avaliação médica.
- A base tem **918 registros** — volume pequeno. Use validação cruzada em vez de
  uma única divisão treino/teste, e não superinterprete diferenças pequenas
  entre modelos.
- Dados coletados entre as décadas de 1980 e 1990, em 4 países. Protocolos
  clínicos e perfil populacional mudaram desde então — o modelo não deve ser
  extrapolado para a população brasileira atual.
- Sem informação temporal — é um retrato no momento do exame, não um
  acompanhamento.
- O alvo é o resultado da angiografia, não a ocorrência de um infarto.

## ✅ Checklist da etapa

- [ ] Pergunta de negócio escrita em uma frase
- [ ] Contexto clínico mínimo compreendido
- [ ] 3 a 6 perguntas de apoio listadas
- [ ] Hipóteses registradas **antes** de olhar os dados
- [ ] Métrica principal e meta numérica definidas
- [ ] Baseline definido
- [ ] Limitações declaradas
- [ ] Conteúdo commitado

```bash
git checkout -b feature/definicao-escopo develop
git add docs/01_entendimento_do_problema.md
git commit -m "docs: define pergunta de negocio, hipoteses e metrica do projeto"
```

➡️ Próxima: [02 — Coleta e fontes de dados](02_coleta_e_fontes_de_dados.md)
