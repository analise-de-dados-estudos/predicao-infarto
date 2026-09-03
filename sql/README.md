# sql/

Consultas SQL do projeto — mesma pasta usada no projeto da PRF.

Use se a base for carregada em um banco (SQLite, PostgreSQL, MySQL) em vez de
lida direto do CSV. É opcional neste projeto, mas costuma valer ponto por
demonstrar domínio da linguagem.

| Arquivo sugerido | Finalidade |
|---|---|
| `01_criacao_tabelas.sql` | DDL: criação das tabelas e tipos |
| `02_carga_dados.sql` | Importação do CSV para a tabela |
| `03_validacoes.sql` | Checagens: nulos, zeros disfarçados, duplicatas, valores fora da faixa |
| `04_consultas_analiticas.sql` | Agregações que respondem às perguntas da etapa 01 |

## Ideias de consultas analíticas

- Taxa de doença cardíaca por faixa etária e sexo
- Proporção de doença por `tipo_dor_peito` e por `inclinacao_st`
- Média de `freq_cardiaca_max` e `depressao_st` por classe do alvo
- Contagem de registros com `colesterol = 0`, cruzada com o alvo — a validação
  que sustenta a decisão da etapa 04

## Padrão

- Palavras-chave em MAIÚSCULAS (`SELECT`, `FROM`, `GROUP BY`)
- Um campo por linha em consultas longas
- Comentário no topo dizendo o que a consulta responde
- Nomes de tabelas e colunas em snake_case, iguais aos da base tratada
