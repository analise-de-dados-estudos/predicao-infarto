# logs/

Registro das **decisões** tomadas em cada etapa. Não é log de execução do
sistema — é o caderno de bordo do projeto.

Por que existe: na hora de escrever o relatório final, você não vai lembrar por
que descartou a coluna `hemisferio` três semanas atrás. Quem anota, escreve o
relatório em um terço do tempo.

## Arquivos sugeridos

| Arquivo | Conteúdo |
|---|---|
| `04_limpeza.md` | O que foi encontrado, o que foi alterado, quantas linhas, por quê |
| `06_features.md` | Variáveis criadas, mantidas e descartadas, com justificativa |
| `07_experimentos.md` | Cada modelo testado, parâmetros e resultado |
| `decisoes.md` | Decisões que mudam o rumo do projeto |

## Modelo de registro

```markdown
## 2026-09-05 — Tratamento da coluna Blood Pressure

**Situação:** coluna vinha como texto "158/88", inutilizável em análise numérica.

**Decisão:** separar em `pressao_sistolica` e `pressao_diastolica` (inteiros)
e criar `pressao_de_pulso` como derivada.

**Motivo:** permite análise numérica e a pressão de pulso tem significado
clínico próprio.

**Impacto:** +3 colunas; nenhuma linha removida; regra sistólica > diastólica
validada em 8.763/8.763 linhas.
```
