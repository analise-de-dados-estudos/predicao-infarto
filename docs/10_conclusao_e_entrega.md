# 10 — Conclusão e entrega

> **Objetivo da etapa:** fechar o projeto — relatório final escrito, repositório
> organizado e versão publicada com tag.

---

## 10.1 Estrutura do relatório final

Salve em `resultados/relatorio_final.md` (ou `.pdf`/`.docx`, conforme o
professor pedir).

| Seção | Conteúdo | Tamanho |
|---|---|---|
| **1. Resumo** | Problema, método, principal resultado, conclusão | 1 parágrafo |
| **2. Introdução** | Doença cardiovascular como problema de saúde pública (use dados do DATASUS), objetivo do trabalho | 1 página |
| **3. Metodologia** | Bases utilizadas, etapas do pipeline, ferramentas, modelos testados | 1–2 páginas |
| **4. Tratamento dos dados** | Achados da limpeza — com destaque para o colesterol zerado | 1 página |
| **5. Análise exploratória** | Principais achados, com os gráficos que os sustentam | 2–3 páginas |
| **6. Modelagem e resultados** | Tabela comparativa, curvas ROC, modelo escolhido e justificativa | 2 páginas |
| **7. Interpretação** | Variáveis mais relevantes e coerência clínica | 1 página |
| **8. Comparação entre bases** | Real × sintética (etapa 12) | 1–2 páginas |
| **9. Limitações** | Volume, época e origem dos dados, viés de seleção, escopo acadêmico | 1 página |
| **10. Conclusão** | Resposta direta à pergunta da etapa 01 | 2 parágrafos |
| **11. Próximos passos** | O que faria com mais tempo ou mais dados | 1 parágrafo |
| **12. Referências** | Bases, artigos, documentação | — |

## 10.2 Como redigir a conclusão

A conclusão precisa responder à pergunta da etapa 01, com número e com ressalva.
Modelo:

> "Sim, é possível. Com as variáveis clínicas disponíveis, o modelo [X] alcançou
> ROC AUC de [valor] (± [desvio]) na validação cruzada, com recall de [valor]
> para a classe de pacientes com doença — desempenho substancialmente superior
> ao baseline trivial (0,50). As variáveis mais determinantes foram
> [ST_Slope, ChestPainType, Oldpeak…], todas provenientes do teste ergométrico,
> o que é coerente com a prática clínica de detecção de isquemia e indica que o
> modelo capturou o fenômeno, e não ruído.
>
> O resultado deve ser lido dentro de seus limites: a base tem 918 registros,
> foi coletada em quatro países entre as décadas de 1980 e 1990, e reúne
> pacientes já encaminhados para angiografia — não a população geral. O modelo
> não é ferramenta diagnóstica e não substitui avaliação médica.
>
> A comparação com a base sintética (seção 8) reforça a conclusão metodológica:
> o mesmo pipeline aplicado a dados sem sinal produz desempenho equivalente ao
> acaso. A qualidade da base, e não a sofisticação do algoritmo, foi o fator
> determinante do resultado."

Três razões pelas quais essa conclusão é forte: responde à pergunta, quantifica
a resposta e declara os limites antes que alguém aponte.

## 10.3 Limitações a declarar (não omita)

- **Volume:** 918 registros é pouco para um problema clínico. A curva de
  aprendizado (etapa 08) provavelmente ainda estava subindo
- **Época:** dados das décadas de 1980–1990. Protocolos e perfil populacional
  mudaram
- **Origem geográfica:** EUA, Hungria e Suíça — não representa a população
  brasileira
- **Viés de seleção:** só pacientes encaminhados a angiografia. As proporções
  não valem para a população geral
- **Dados ausentes:** ~19% dos colesteróis eram zeros disfarçados; a imputação
  introduz incerteza
- **Desbalanceamento por sexo:** conclusões sobre mulheres são menos confiáveis
- **Alvo:** é o resultado da angiografia, não a ocorrência de infarto

## 10.4 Checklist do repositório

- [ ] `README.md` completo: objetivo, estrutura, como executar, fonte dos dados
- [ ] Todos os notebooks executam de ponta a ponta, sem erro, em ordem
- [ ] Notebooks salvos **com as saídas visíveis** (gráficos renderizados)
- [ ] Nenhum caminho absoluto do seu computador (`C:\Users\...`) no código
- [ ] `requirements.txt` atualizado (`pip freeze`)
- [ ] Nenhuma credencial versionada (`kaggle.json`, senhas, tokens)
- [ ] Pastas vazias com `.gitkeep`
- [ ] Todos os arquivos de `docs/` preenchidos
- [ ] Histórico de commits seguindo o padrão
- [ ] `main` com a versão estável e uma tag de versão

## 10.5 Teste de reprodutibilidade

Peça a um colega — ou faça você mesmo em outra pasta:

```bash
git clone <url-do-repositorio> teste-reproducao
cd teste-reproducao
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# rodar os notebooks de 01 a 07, na ordem
```

Se algum passo falhar ou exigir um arquivo que não está no repositório, o
projeto **não é reprodutível** — corrija antes de entregar. Se a base bruta não
estiver versionada, o `dados_brutos/README.md` precisa trazer o comando exato de
download. Este é o teste que mais reprova projeto bom.

## 10.6 Publicação da versão

```bash
# 1. Fechar a release
git checkout -b release/v1.0.0 develop
# ajustes finais de texto e versão
git commit -m "docs: revisa relatorio final e atualiza README para a v1.0.0"

# 2. Levar para main
git checkout main
git merge --no-ff release/v1.0.0
git tag -a v1.0.0 -m "Versao 1.0.0 - projeto de predicao de doenca cardiaca"
git push origin main --tags

# 3. Devolver os ajustes para develop
git checkout develop
git merge --no-ff release/v1.0.0
git push origin develop

# 4. Remover a branch de release
git branch -d release/v1.0.0
```

## 10.7 Apresentação (se houver)

Sugestão de 10 slides, 10 a 15 minutos:

1. Problema e pergunta de negócio
2. As bases: origem, volume, variáveis — e por que duas
3. Metodologia (o fluxo do dado)
4. Limpeza: o colesterol zerado e o que ele revela
5. EDA: dois ou três gráficos mais reveladores
6. Modelos testados
7. Resultados: tabela comparativa + curva ROC
8. Interpretação: as variáveis do teste ergométrico no topo do ranking
9. Comparação real × sintética: o painel de ROCs lado a lado
10. Conclusões, limitações e próximos passos

Perguntas prováveis da banca e onde a resposta está:

| Pergunta | Slide |
|---|---|
| "Por que não usou a base indicada?" | 9 — usou, como controle |
| "Por que a acurácia não é a métrica principal?" | 7 |
| "O modelo pode ser usado num hospital?" | 10 — limitações |
| "Por que o colesterol não é importante?" | 4 e 5 |

## ✅ Checklist final

- [ ] Relatório final escrito e revisado
- [ ] Limitações declaradas
- [ ] Repositório limpo e reprodutível
- [ ] Tag de versão criada e publicada
- [ ] `main` atualizada
- [ ] Apresentação pronta (se aplicável)
- [ ] Link do repositório enviado ao professor

⬅️ Consulta contínua: [11 — Git Flow e padrão de commit](11_git_flow_e_padrao_de_commit.md)
