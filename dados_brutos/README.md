# dados_brutos/

Pasta dos **dados originais, exatamente como foram baixados da fonte**.

## 🔒 Regras desta pasta

1. **Nenhum arquivo aqui é alterado.** Nunca. Nem para corrigir um acento.
2. Todo tratamento é feito em memória (notebook) e salvo em `dados_tratados/`.
3. Se a base for atualizada na fonte, o arquivo novo entra com sufixo de data
   (ex.: `heart_2026-09.csv`) — o antigo permanece.
4. Toda base aqui precisa estar descrita na tabela abaixo.

> **Por quê?** Se alguém (ou você daqui a três meses) clonar o repositório e
> rodar os notebooks na ordem, tem que chegar exatamente no mesmo resultado.
> Isso só funciona se o ponto de partida for imutável.

## Arquivos

| Arquivo | Fonte | Data do download | Linhas | Colunas | Natureza | Papel |
|---|---|---|---|---|---|---|
| `heart.csv` | [Kaggle — Heart Failure Prediction](https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction) | *preencher* | 918 | 12 | **Real** | Base principal |
| `heart_attack_prediction_dataset.csv` | [Kaggle — Heart Attack Risk Prediction](https://www.kaggle.com/datasets/iamsouravbanerjee/heart-attack-prediction-dataset) | *preencher* | 8.763 | 26 | Sintética | Contraprova |

---

### heart.csv — base principal

- **Autor:** fedesoriano
- **Origem:** consolidação de 5 bases clínicas do repositório UCI — Cleveland
  (303), Hungria (294), Suíça (123), Long Beach VA (200) e Statlog (270).
  Total de 1.190 registros, dos quais 272 eram duplicatas → **918 registros
  únicos**
- **Licença:** ver a página do Kaggle (aba *Data*) e registrar aqui
- **Separador:** vírgula (`,`) · **Encoding:** UTF-8 · **Decimal:** ponto (`.`)
- **Variável alvo:** `HeartDisease` (0 = sem doença, 1 = com doença)
- **Pontos de atenção conhecidos** (confirme ao abrir o arquivo):
  - `Cholesterol = 0` em ~172 registros → **valor ausente disfarçado de zero**
  - `RestingBP = 0` em 1 registro → idem
  - `Oldpeak` com valores negativos → verificar plausibilidade

⚠️ **Não confunda com a base [johnsmith88/heart-disease-dataset](https://www.kaggle.com/datasets/johnsmith88/heart-disease-dataset)** (1.025 linhas).
Aquela é a Cleveland com linhas duplicadas; modelos treinados nela atingem 99%
de acurácia por vazamento de dados, não por aprendizado. Ao abrir qualquer
base, rode `df.duplicated().sum()` antes de comemorar métrica.

---

### heart_attack_prediction_dataset.csv — base secundária

- **Autor:** Sourav Banerjee
- **Natureza:** **sintética** — os valores foram gerados artificialmente e não
  descrevem pacientes reais
- **Variável alvo:** `Heart Attack Risk` (0 / 1), distribuição 64% / 36%
- **Papel no projeto:** experimento de controle. O mesmo pipeline é aplicado a
  ela para demonstrar que a diferença de resultado vem dos dados, não do método
- **Verificado no arquivo:** zero nulos, zero duplicatas, e a maior correlação
  absoluta entre qualquer variável e o alvo é **0,019**

O dicionário completo das duas bases está em
`docs/03_dicionario_de_variaveis.md`.
