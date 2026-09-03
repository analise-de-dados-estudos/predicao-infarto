# resultados/

Tudo que o projeto **gera** e que entra no relatório final.

| Subpasta | Conteúdo | Formato |
|---|---|---|
| `figuras/` | Gráficos exportados da EDA, curvas ROC, importância de variáveis | `.png` (≥150 dpi) ou `.svg` |
| `tabelas/` | Tabelas resumo, estatísticas descritivas, tabelas cruzadas | `.csv` / `.xlsx` |
| `metricas/` | Métricas dos modelos e comparativo | `.csv` / `.json` |
| `modelos/` | Modelos treinados serializados | `.joblib` / `.pkl` |

## Padrão de nomes

`<etapa>_<descricao_curta>.<ext>` — minúsculas, sem acento, com underscore.

```
figuras/05_distribuicao_idade.png
figuras/05_taxa_risco_por_faixa_etaria.png
figuras/05_matriz_correlacao.png
figuras/08_curva_roc_comparativa.png
figuras/09_importancia_variaveis.png
tabelas/05_estatisticas_descritivas.csv
metricas/comparativo_modelos.csv
modelos/random_forest_v1.joblib
```

## Regras

- Toda figura do relatório precisa existir aqui — nada de print de tela
- Arquivo gerado por notebook é **descartável**: tem que ser possível apagar
  tudo e recriar rodando os notebooks na ordem
- Modelos `.joblib` grandes estão no `.gitignore`; se precisar versionar,
  remova a regra correspondente e justifique
