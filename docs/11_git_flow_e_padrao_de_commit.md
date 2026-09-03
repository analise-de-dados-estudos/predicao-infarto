# 11 — Git Flow e padrão de commit

> Documento de consulta contínua. Leia antes do primeiro commit e volte sempre
> que iniciar uma etapa nova.

---

## 11.1 O que é Git Flow

Um modelo de organização de branches criado por Vincent Driessen. Em vez de
todos os commits caírem em uma única linha do tempo, o trabalho é separado por
finalidade.

| Branch | Vida | Origem | Destino | Para que serve |
|---|---|---|---|---|
| `main` | permanente | — | — | Versão estável e entregável. Só recebe merge de `release/*` ou `hotfix/*` |
| `develop` | permanente | `main` | — | Integração do trabalho em andamento |
| `feature/*` | temporária | `develop` | `develop` | Uma etapa/funcionalidade por branch |
| `release/*` | temporária | `develop` | `main` + `develop` | Preparação da entrega |
| `hotfix/*` | temporária | `main` | `main` + `develop` | Correção urgente na versão publicada |

```
main      ──●─────────────────────────●────────► v1.0.0
             \                       /
develop    ───●───●───────●─────────●──────────►
                  \      /  \      /
feature/eda        ●────●    \    /
feature/modelagem             ●──●
```

## 11.2 Criação do repositório

```bash
mkdir predicao-infarto && cd predicao-infarto
git init
git branch -M main

# estrutura + arquivos base
git add .
git commit -m "chore: cria estrutura inicial de pastas do projeto"

# repositório remoto (crie antes no GitHub, sem README)
git remote add origin https://github.com/<seu-usuario>/predicao-infarto.git
git push -u origin main

# branch de desenvolvimento
git checkout -b develop
git push -u origin develop
```

> No GitHub, em **Settings → Branches**, defina `develop` como branch padrão
> durante o desenvolvimento e proteja a `main` contra push direto.

## 11.3 Ciclo de uma etapa (o que você repete 10 vezes)

```bash
# 1. Partir de develop atualizada
git checkout develop
git pull origin develop

# 2. Criar a branch da etapa
git checkout -b feature/analise-exploratoria

# 3. Trabalhar, com commits pequenos e frequentes
git add notebooks/03_analise_exploratoria.ipynb
git commit -m "feat(eda): adiciona analise univariada das variaveis numericas"

git add resultados/figuras/
git commit -m "feat(eda): exporta graficos de distribuicao por faixa etaria"

# 4. Publicar a branch
git push -u origin feature/analise-exploratoria

# 5. Abrir Pull Request no GitHub: feature/... -> develop
#    Revisar, então fazer merge

# 6. Limpar
git checkout develop
git pull origin develop
git branch -d feature/analise-exploratoria
```

## 11.4 Nomes de branch

Padrão: `tipo/descricao-curta-com-hifen`, minúsculas, sem acento.

| ✅ Bom | ❌ Ruim |
|---|---|
| `feature/limpeza-dados` | `feature/Limpeza de Dados` |
| `feature/analise-exploratoria` | `nova-branch` |
| `feature/modelo-random-forest` | `teste2` |
| `fix/grafico-correlacao` | `correcao` |
| `release/v1.0.0` | `versao-final-FINAL` |

Branches deste projeto, na ordem: `feature/definicao-escopo`,
`feature/coleta-dados`, `feature/dicionario-dados`, `feature/limpeza-dados`,
`feature/analise-exploratoria`, `feature/engenharia-atributos`,
`feature/modelagem`, `feature/avaliacao-modelos`,
`feature/interpretabilidade`, `feature/comparacao-bases`, `release/v1.0.0`.

## 11.5 Padrão de commit — Conventional Commits

```
<tipo>(<escopo opcional>): <descrição no imperativo>

[corpo opcional: explica o PORQUÊ, não o o quê]

[rodapé opcional: refs #12]
```

### Tipos

| Tipo | Uso | Exemplo |
|---|---|---|
| `feat` | Nova análise, notebook, modelo, gráfico | `feat(eda): adiciona matriz de correlacao` |
| `fix` | Correção de erro | `fix(limpeza): corrige conversao de pressao para inteiro` |
| `docs` | Documentação | `docs: atualiza README com estrutura de pastas` |
| `data` | Bases de dados | `data: adiciona base bruta do Kaggle` |
| `refactor` | Reorganização sem mudar resultado | `refactor: extrai funcao de padronizacao de nomes` |
| `perf` | Desempenho | `perf: otimiza leitura do csv com dtypes explicitos` |
| `test` | Testes e validações | `test: valida regras de consistencia da base tratada` |
| `chore` | Manutenção do projeto | `chore: atualiza requirements.txt` |
| `style` | Formatação | `style: padroniza identacao dos notebooks` |

### Regras

1. **Imperativo:** "adiciona", nunca "adicionado" ou "adicionando"
2. **Minúscula** no início da descrição
3. **Sem ponto final**
4. **Máximo 72 caracteres** na primeira linha
5. **Um assunto por commit** — se precisar escrever "e", provavelmente são dois
6. Sem acentos (evita problemas de encoding em terminais antigos)
7. O corpo explica o **porquê**, não o que (o diff já mostra o que)

### Exemplo de commit com corpo

```
feat(limpeza): trata colesterol zerado como valor ausente

172 registros (19% da base) tinham Cholesterol = 0, valor
fisiologicamente impossivel: sao exames nao realizados registrados
como zero. Substituidos por nulo, imputados pela mediana e sinalizados
na coluna indicadora colesterol_ausente, preservando as 918 linhas.
```

### Frequência

- ❌ Um commit gigante no fim do dia: `"projeto pronto"`
- ✅ Commits a cada bloco concluído: 3 a 8 por etapa

Rode `git status` e `git diff` antes de cada `git add` — o commit sai melhor
quando você sabe exatamente o que está indo.

## 11.6 Pull Requests

Mesmo trabalhando sozinho, abra PR: cria histórico, obriga a revisar o próprio
trabalho e fica visível para o professor.

**Modelo de descrição:**

```markdown
## O que foi feito
Análise exploratória completa das 12 variáveis da base clínica.

## Etapa do manual
docs/05_analise_exploratoria.md

## Principais resultados
- ST_Slope e ChestPainType são as variáveis mais associadas ao diagnóstico
- MaxHR é significativamente menor no grupo com doença (p < 0,001)
- Colesterol discrimina pouco — H6 confirmada
- Hipóteses H1 a H5 confirmadas

## Arquivos alterados
- notebooks/03_analise_exploratoria.ipynb
- resultados/figuras/ (12 gráficos)

## Checklist
- [x] Notebook executa do início ao fim sem erro
- [x] Gráficos exportados
- [x] Achados documentados
```

## 11.7 Versionamento semântico (tags)

`MAJOR.MINOR.PATCH`

| Parte | Quando incrementar | Exemplo |
|---|---|---|
| MAJOR | Mudança que quebra compatibilidade / refaz o projeto com outra base | 2.0.0 |
| MINOR | Nova análise ou modelo | 1.1.0 |
| PATCH | Correção pontual | 1.0.1 |

```bash
git tag -a v1.0.0 -m "Primeira versao completa do projeto"
git push origin --tags
```

Sugestão para este projeto: `v0.1.0` (estrutura + dados), `v0.5.0` (EDA
concluída), `v1.0.0` (entrega final).

## 11.8 Cuidados com dados e notebooks no Git

| Situação | O que fazer |
|---|---|
| Arquivo > 100 MB | O GitHub rejeita. Use Git LFS ou deixe fora do repositório, documentando o download |
| Base bruta média (< 50 MB) | Pode versionar — é o caso deste projeto (`heart.csv` tem ~24 KB; a base sintética, 1,4 MB) |
| Notebook com saídas pesadas | Ocupa espaço e gera conflitos difíceis. Considere `nbstripout` |
| Conflito em `.ipynb` | Formato JSON — resolver na mão é penoso. Prefira reexecutar a partir da versão correta |
| Credenciais | **Nunca.** Se acontecer, troque a chave imediatamente — remover o commit não basta |

## 11.9 Comandos de emergência

```bash
# desfazer o último commit, mantendo as alterações
git reset --soft HEAD~1

# descartar alterações de um arquivo (irreversível)
git checkout -- caminho/arquivo.py

# corrigir a mensagem do último commit (antes do push)
git commit --amend -m "nova mensagem"

# ver o histórico resumido
git log --oneline --graph --all --decorate

# guardar alterações temporariamente para trocar de branch
git stash
git stash pop

# reverter um commit já publicado (cria um commit de reversão)
git revert <hash>
```

⚠️ Nunca use `git push --force` em branch compartilhada.

## 11.10 Checklist antes de cada push

- [ ] `git status` limpo, sem arquivos indesejados
- [ ] Nenhuma credencial ou arquivo temporário incluído
- [ ] Mensagem de commit no padrão
- [ ] Notebook executa sem erro
- [ ] Está na branch correta (`git branch --show-current`)

⬅️ Voltar para a [visão geral](00_visao_geral.md)
