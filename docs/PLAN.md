# PLAN.md — Construção do notebook (documento histórico)

> ## ⚠️ Superado em 2026-08-18 nas seções §2 e §5
>
> Este plano foi escrito para uma planilha que não existe mais e para um desenho
> que os dados não sustentam: EPTs em saliva por participante e comparação
> exposto × controle em biomarcador.
>
> **A fonte de escopo científico vigente é**
> `docs/superpowers/specs/2026-08-17-analise-citogenetica-design.md`.
> **O plano de implementação vigente é**
> `docs/superpowers/plans/2026-08-18-pipeline-citogenetica.md`.
>
> O que mudou, em resumo:
>
> | Este documento diz | Vale hoje |
> |---|---|
> | 16 variáveis analíticas (7 marcadores + 9 EPTs) | 7 marcadores; **nenhum EPT em saliva** |
> | `data/dados_mock.xlsx`, aba `dados` | `planilha_banco_de_dados.xlsx`, aba `SEM SALIVA` |
> | Mann-Whitney exposto × controle nos marcadores | Fisher e Kruskal-Wallis **entre as seis escolas** |
> | 120 pares no Spearman | 10 pares |
> | Bloco 12.5 como *stub* | calculado, com o cromo de Elpídio Paranhos |
> | notebook primário `analise_pesquisa.ipynb` | **excluído**; a variante é a fonte da verdade |
>
> As seções §1, §3, §4 e §6–§9 continuam úteis como registro de como o projeto
> foi construído. **§2 e §5 não devem ser usadas como referência.**

---
## Cabeçalho original

> Plano de execução da pipeline analítica descrita em `CLAUDE.md`, com o nível de
> rigor estatístico exigido por `planning/spec_artigo_referencia.md` e com os
> invariantes científicos de `planning/RESEARCH_INVARIANTS.md`.
>
> Este documento é o **mapa de construção**. Ele responde a três perguntas:
> 1. **Em que ordem** os blocos serão construídos.
> 2. **O que cada bloco precisa entregar** para ser considerado pronto.
> 3. **Como verificar** que cada bloco está correto antes de seguir adiante.

---

## 0. Estado atual e premissas

- A planilha real **ainda não chegou**. A pesquisadora vai usar o modelo
  versionado em `data/dados_mock.xlsx` como referência de estrutura para
  preencher a planilha real.
- O escopo estatístico segue o **rigor pleno** do `spec_artigo_referencia.md`
  (não a versão minimalista do CLAUDE.md). Tamanho de efeito, correção de
  múltiplas comparações, clustering hierárquico do heatmap, IC bootstrap e
  biplot anotado **fazem parte do escopo**.
- Idioma do notebook (markdown e comentários): **PT-BR**.
- Filosofia: célula curta, responsabilidade única, código explícito, sem
  abstrações prematuras (`CLAUDE.md` §"Filosofia de implementação"; também
  `BEHAVIORAL_GUIDELINES.md` §2 e §3).

---

## 1. Estrutura de pastas alvo

```
Tese da Brendinha/
├── CLAUDE.md
├── BEHAVIORAL_GUIDELINES.md
├── requirements.txt
├── bug_fix/                     # registros de bugs (vazio até o primeiro)
├── data/
│   ├── dados_mock.xlsx          # planilha sintética versionada (modelo)
│   └── dados_reais.xlsx         # quando chegar (NÃO versionar)
├── geradores/                   # scripts geradores — versionados, travados
│   ├── construir_notebook.py
│   └── gerar_dados_mock.py
├── notebook/
│   └── analise_pesquisa.ipynb
├── outputs/
│   ├── tabelas/                 # CSVs dos resultados
│   └── figuras/                 # PNGs dos plots
└── planning/
    ├── PLAN.md
    ├── PROJECT_BUILDING.md
    ├── RESEARCH_INVARIANTS.md
    └── spec_artigo_referencia.md
```

`data/dados_reais.xlsx` deve entrar em `.gitignore` quando chegar (dados de
pesquisa com sujeitos humanos não vão para o repositório).

---

## 2. Conteúdo da planilha mock (`data/dados_mock.xlsx`)

A planilha mock serve **dois propósitos**: (i) referência de estrutura para a
pesquisadora e (ii) entrada para validar a pipeline ponta-a-ponta. Deve
conter:

**Aba única**: `dados`

**Colunas de identificação e grupo**
- `id_participante` — string, único por linha
- `grupo` — categórica binária: `exposto` | `controle`

> **Origem do `grupo` (corrigido em 2026-06-07).** Na mock, `grupo` já vem
> rotulada. No banco real `BANCO DE DADOS BeG.xlsx`, ela é **derivada da
> instituição de coleta** (primeira coluna, sem cabeçalho): `UFBA` → `controle`,
> demais instituições → `exposto`, descartando as linhas de status
> (`sem saliva`, `tubo sumido`, etc.). Regra canônica em
> `planning/RESEARCH_INVARIANTS.md` § "Grupo de exposição".

**Colunas de marcadores citogenéticos** (contagens absolutas por 2.000 células,
conforme protocolo do artigo de referência)
- `micronucleos`
- `binucleadas`
- `brotamento_nuclear`
- `cariorrexis`
- `cariolise`
- `picnose`
- `celulas_normais`

**Variáveis de exposição — EPTs** (em µg/L, separadas por matriz biológica e
ambiental conforme `RESEARCH_INVARIANTS.md`)

EPTs em saliva (4):
- `saliva_As`, `saliva_Cd`, `saliva_Pb`, `saliva_Hg`

EPTs em água (5):
- `agua_As`, `agua_Cd`, `agua_Pb`, `agua_Hg`, `agua_Cr`

**Variáveis de localização** (9 colunas, conforme `RESEARCH_INVARIANTS.md`)
- `local_moradia`, `bairro_moradia`, `municipio_moradia`
- `escola`, `bairro_escola`, `municipio_escola`
- `zona_exposicao`, `ponto_coleta_agua`, `fonte_agua`

**Fatores de confusão** (16 colunas, conforme `RESEARCH_INVARIANTS.md`) —
**registrados no schema para uso futuro**; nesta entrega são apenas validados
quanto à presença, sem checagem de tipo nem uso analítico
- `idade`, `sexo`, `tempo_residencia`
- `habitos_alimentares`, `consumo_peixe`, `consumo_agua_torneira`,
  `consumo_agua_filtrada`, `uso_poco_artesiano`
- `higienizacao_bucal_frequencia`, `uso_enxaguante_bucal`
- `uso_tabaco_passivo`, `exposicao_ocupacional_familiar`
- `uso_medicamentos`, `condicoes_saude_relevantes`
- `renda_familiar`, `escolar`

**Tamanho sugerido**: 40 linhas (20 expostos, 20 controle), com distribuições
plausíveis (assimétricas, com algum zero em marcadores raros). Os números são
fictícios, apenas para a pipeline rodar.

> A planilha mock foi gerada uma vez por `geradores/gerar_dados_mock.py` e
> versionada como ponto fixo. O gerador fica versionado como referência mas
> está **travado** (env var `EXECUTAR_GERADOR=1` exigida) — a pesquisadora
> ajusta colunas diretamente na planilha quando a planilha real exigir
> adaptação. Ver `CLAUDE.md` "Comandos comuns".

---

## 3. Decisões de arquitetura do notebook

### 3.1. Configuração centralizada (Bloco 2)
Todas as variáveis que mudam entre rodadas vivem em **um único bloco de
configuração**:

- caminho da planilha (`data/dados_mock.xlsx` por padrão)
- nome da aba
- nome da coluna de grupo + rótulos esperados
- listas explícitas: `COLUNAS_ID_GRUPO`, `COLUNAS_MARCADORES`,
  `COLUNAS_EPTS_SALIVA`, `COLUNAS_EPTS_AGUA`, `COLUNAS_LOCALIZACAO`,
  `COLUNAS_CONFOUNDERS`
- parâmetros estatísticos: nível de significância, método de correção de
  múltiplas comparações, número de bootstraps, seed aleatória, método de
  ligação do clustering

Trocar para a planilha real = trocar **uma string**. Esse é o teste de
ergonomia do Bloco 2.

### 3.2. Sem classes, com funções pequenas
- Funções com nome literal (`calcular_mann_whitney_por_variavel`,
  `aplicar_correcao_bh`, etc.).
- Cada função faz uma coisa. Onde uma transformação aparece em duas células,
  vira função; onde aparece uma vez só, fica inline (`BEHAVIORAL_GUIDELINES.md`
  §2).
- Evitar `df`, `df2`, `temp`, `dados_ok`. Nomes refletem conteúdo: `dados_brutos`,
  `dados_padronizados`, `tabela_descritiva`, `resultado_mw`, `matriz_spearman`.

### 3.3. Cada bloco mostra antes-e-depois
Markdown curto explica **o que** e **por quê**; célula seguinte mostra preview
de entrada; célula seguinte calcula; célula seguinte mostra preview de saída
(`CLAUDE.md` §"Regras pedagógicas").

### 3.4. Exportação no Bloco 13, não espalhada
Cada bloco deixa seu artefato em variável Python; **só o Bloco 13** escreve em
disco. Isso mantém os blocos analíticos puros e facilita re-execução parcial.

---

## 4. Ordem de construção e marcos verificáveis

Cada fase produz algo verificável **antes** de seguir adiante. Se um marco
falha, paramos e corrigimos — não acumulamos blocos quebrados.

### Fase A — Fundação (Blocos 0, 1, 2, 13 esqueleto)
**Construir**: capa, imports, configuração, esqueleto de exportação.
**Verificar**: notebook abre, imports rodam, configuração está em uma única
célula legível, paths existem.

### Fase B — Ingestão e validação (Blocos 3, 4, 5)
**Construir**: leitura do Excel, padronização de nomes de coluna, validação de
estrutura.
**Verificar**: rodar contra `dados_mock.xlsx` mostra shape esperado, todas as
colunas obrigatórias presentes, tipos coerentes; rodar contra um Excel
deliberadamente quebrado dispara mensagens de erro claras (coluna faltando,
aba errada).

### Fase C — Preparação analítica (Blocos 6, 7, 8)
**Construir**: separação de variáveis, coerção numérica, dataset analítico
final.
**Verificar**: contagem de NaN antes/depois é exibida; relatório simples lista
quais células viraram NaN; `dataset_analitico` tem só as colunas esperadas e
todas numéricas onde precisa.

### Fase D — Estatística descritiva (Bloco 9)
**Construir**: tabela `média ± DP`, `mediana (IQR)`, `min`, `max`, **estratificada
por grupo**, para marcadores e para EPTs (saliva + água). EPTs também
estratificados por **três variáveis espaciais**: `escola`, `zona_exposicao` e
`bairro_moradia` (preserva granularidade espacial — spec §4).
**Verificar**: tabela legível, sem células vazias, números batem com
`describe()` aplicado manualmente em uma variável de teste.

### Fase E — Inferência univariada (Bloco 10)
**Construir**: Mann-Whitney bilateral por marcador e por EPT (saliva e água,
totalizando 7 + 9 = 16 variáveis); reportar estatística U, *p* bruto, *p*
ajustado por **Benjamini-Hochberg**, tamanho de efeito (`r = Z/√N` **e** `A`
de Vargha-Delaney).
**Verificar**: tabela consolidada com uma linha por variável; *p* ajustados
≥ *p* brutos; tamanhos de efeito dentro de faixas plausíveis ([0,1] para `A`).

### Fase F — Associação por postos (Bloco 11)
**Construir**: matriz Spearman entre marcadores e EPTs (16 variáveis = 120
pares únicos); *p*-valores com correção BH; ICs bootstrap (95%, B = 1.000)
para os pares mais relevantes; heatmap **reordenado por clustering
hierárquico aglomerativo** sobre `1 - |ρ|`, com método de ligação
parametrizável (default: *average*).
**Verificar**: matriz simétrica com diagonal = 1; heatmap mostra blocos
visualmente coerentes; reordenação muda quando o método de ligação muda.

### Fase G — Multivariada exploratória (Bloco 12)
**Construir**: padronização z-score, PCA sobre marcadores + EPTs (16
variáveis ativas), scree plot, tabela de loadings, **biplot** com vetores
de loadings e scores coloridos por grupo; reportar variância explicada
cumulativa e justificar o número de componentes retidos.
**Verificar**: variância acumulada das componentes retidas ≥ 60% (ou
justificativa explícita); biplot tem eixos rotulados com `% variância`.

### Fase H — Contextualização regulatória (Bloco 12.5)
**Construir**: tabela MPV-por-elemento (referências OMS, USEPA, UE) lado a
lado com concentrações observadas; razão `observado / MPV` por amostra.
**Marco condicional**: a **tabela de MPVs precisa ser fornecida pela
orientação científica** (quais agências, quais limites, qual matriz). Até lá,
o bloco fica como *stub* documentado, com a estrutura pronta e um placeholder
explícito (`# TODO: substituir tabela MPV após decisão científica`). Não
inventamos limites regulatórios.
**Verificar**: quando os MPVs forem providos, o bloco produz a tabela
comparativa sem alterações de código além do CSV/dict de limites.

### Fase I — Exportação (Bloco 13 completo)
**Construir**: salvar tabela limpa, descritiva, MW, Spearman, loadings PCA,
figuras (heatmap reordenado, biplot, scree). Nomes previsíveis em
`outputs/tabelas/` e `outputs/figuras/`.
**Verificar**: rodar o notebook do topo ao fim em um diretório limpo recria
todos os artefatos.

### Fase J — Notas para tese (Bloco 14)
**Construir**: markdown com frases curtas registrando decisões metodológicas
(escolha do quadro não-paramétrico, justificativa do Spearman, escolha do
método de ligação, motivo de excluir variáveis demográficas da PCA ativa,
limitações do desenho transversal — espelhando seções 3, 6.1, 7.2 e 11 da
spec).
**Verificar**: cada decisão metodológica relevante do spec tem uma frase
correspondente no Bloco 14.

---

## 5. Detalhamento bloco-a-bloco

### Bloco 0 — Capa e objetivo
Markdown com: título, objetivo, entrada esperada (`data/dados_mock.xlsx`
enquanto a planilha real não chega), análises produzidas, autor, data.

### Bloco 1 — Imports
Apenas `pandas`, `numpy`, `scipy.stats`, `sklearn.decomposition.PCA`,
`sklearn.preprocessing.StandardScaler`, `scipy.cluster.hierarchy`,
`statsmodels.stats.multitest`, `matplotlib.pyplot`, `seaborn`, `pathlib.Path`.
Nada além.

### Bloco 2 — Configuração
Variáveis exportadas (todas em um lugar):
- `CAMINHO_PLANILHA`, `NOME_ABA`
- `COLUNA_GRUPO`, `ROTULOS_GRUPO = ("exposto", "controle")`
- `COLUNAS_ID_GRUPO = ["id_participante", "grupo"]`
- `COLUNAS_MARCADORES = [...]` (lista fixa de 7 marcadores)
- `COLUNAS_EPTS_SALIVA = ["saliva_As", "saliva_Cd", "saliva_Pb", "saliva_Hg"]`
- `COLUNAS_EPTS_AGUA = ["agua_As", "agua_Cd", "agua_Pb", "agua_Hg", "agua_Cr"]`
- `COLUNAS_LOCALIZACAO = [...]` (lista fixa de 9 variáveis)
- `COLUNAS_CONFOUNDERS = [...]` (lista fixa de 16 variáveis — registradas
  para uso futuro, sem checagem de tipo nesta entrega)
- `ALPHA = 0.05`, `METODO_CORRECAO = "fdr_bh"`, `BOOTSTRAP_N = 1000`,
  `SEED = 42`, `LIGACAO_CLUSTER = "average"`
- `DIR_OUTPUTS_TABELAS`, `DIR_OUTPUTS_FIGURAS`

### Bloco 3 — Leitura
`pd.read_excel(CAMINHO_PLANILHA, sheet_name=NOME_ABA)`. Mostra `shape`,
`head()`, lista de abas disponíveis (útil quando a planilha real chegar).

### Bloco 4 — Padronização das colunas
Função `padronizar_nomes_colunas(df)`: minúsculas, troca espaço por `_`, remove
acentos, remove caracteres não-alfanuméricos. Mostra `before → after`.

### Bloco 5 — Validação inicial
- Conferir existência de **todas** as colunas esperadas, agrupadas por
  categoria (id+grupo, marcadores, EPTs saliva, EPTs água, localização,
  confounders). Mensagem de erro lista colunas faltantes **por categoria**
  para diagnóstico rápido.
- Conferir não-duplicidade de `id_participante`.
- Conferir tipos numéricos para marcadores + EPTs (saliva e água).
  **Confounders não passam por checagem de tipo** — apenas presença é
  validada (registrados para uso futuro).
- Conferir que `grupo` só assume valores em `ROTULOS_GRUPO`.
- Falha = mensagem clara apontando a coluna/linha.

### Bloco 6 — Separação das variáveis
Cria objetos explícitos: `dados_id`, `dados_marcadores`, `dados_epts_saliva`,
`dados_epts_agua`, `dados_localizacao`, `dados_confounders`. Imprime quantas
variáveis foram alocadas em cada bucket. Tornar explícito é a regra
pedagógica.

### Bloco 7 — Limpeza e coerção
Para marcadores e EPTs (saliva + água): `pd.to_numeric(errors='coerce')`.
Função `relatorio_de_coercao(df_antes, df_depois)` lista, por coluna, quantos
valores viraram NaN. Localização e confounders **não passam por coerção** —
ficam como vieram da planilha.

### Bloco 8 — Dataset analítico final
`dataset_analitico = pd.concat([dados_id, dados_marcadores,
dados_epts_saliva, dados_epts_agua, dados_localizacao, dados_confounders],
axis=1)`, sem `dropna` global (a política de NaN fica explícita por
análise — descritiva/MW usam pares disponíveis, Spearman pares completos por
par, PCA listwise; spec §6.2 e ADVERSARIAL_REVIEW.md §29). Exibe shape final
e preview.

### Bloco 9 — Estatística descritiva
Função `descritiva_por_grupo(df, variaveis, agrupador)` retorna DataFrame com
`n, média, DP, mediana, IQR, min, max` por grupo. Aplicar a:
- marcadores por grupo;
- EPTs (saliva + água, com coluna `matriz` derivada do prefixo) por grupo;
- EPTs por `escola`;
- EPTs por `zona_exposicao`;
- EPTs por `bairro_moradia`.

(spec §4 — preservar granularidade espacial em três eixos de localização.)

### Bloco 10 — Mann-Whitney
- Função `mann_whitney_por_variavel(df, variavel, grupo)` retorna `U`, `p`,
  `r`, `A_vargha_delaney`.
- Loop sobre marcadores + EPTs (saliva + água) = 16 variáveis → DataFrame
  consolidado.
- Aplicar `multipletests(p, method=METODO_CORRECAO)` para obter `p_ajustado`.
- Tabela final ordenada por `p_ajustado`, com coluna `classe`
  (`marcador`/`saliva`/`agua`).

### Bloco 11 — Spearman + clustering
- `matriz_rho, matriz_p = spearmanr(matriz_numerica)` sobre marcadores +
  EPTs (16 variáveis, 120 pares únicos).
- Achatar pares, aplicar correção BH aos *p*-valores.
- Para os top-K pares por `|rho|`, calcular IC bootstrap 95%.
- Distância `D = 1 - |rho|` → `linkage(D_condensada, method=LIGACAO_CLUSTER)`
  → reordenar índices → heatmap reordenado.

### Bloco 12 — PCA
- `StandardScaler().fit_transform(matriz_numerica)` sobre marcadores + EPTs
  (16 variáveis ativas).
- `PCA(n_components=min(n,p))`. Reportar `explained_variance_ratio_` cumulada.
- Tabela de loadings (componentes × variáveis).
- Biplot: scatter de scores nos PC1×PC2 colorido por `grupo`, vetores de
  loadings sobrepostos com rótulos.

### Bloco 12.5 — Contextualização regulatória *(stub até decisão científica)*
- Estrutura: dicionário/CSV `mpv_por_elemento` com colunas
  `[elemento, agencia, limite, unidade]`.
- Cálculo: para cada amostra e cada elemento, razão `observado / limite`.
- Saída: tabela longa + mini-heatmap de razões.
- Marca explícita no markdown: bloco **inativo** até a tabela de MPVs ser
  fornecida pela orientação.

### Bloco 13 — Exportação
- `dataset_analitico.csv`, `descritiva_marcadores.csv`,
  `descritiva_epts.csv`, `descritiva_epts_por_escola.csv`,
  `descritiva_epts_por_zona_exposicao.csv`,
  `descritiva_epts_por_bairro_moradia.csv`,
  `mann_whitney.csv`, `spearman_pares.csv`,
  `spearman_top_pares_bootstrap.csv`, `spearman_rho_matriz.csv`,
  `spearman_p_matriz.csv`, `pca_loadings.csv`, `pca_variancia.csv`.
- Figuras: `heatmap_spearman.png`, `biplot_pca.png`, `scree_pca.png`.
- Função `salvar_tabela(df, nome)` e `salvar_figura(fig, nome)` centralizam
  caminho e formato.

### Bloco 14 — Notas para tese
Markdown com frases curtas, uma por decisão metodológica. Espelha as seções
3, 5, 6, 7, 11 da spec. Servirá de matéria-prima para o capítulo de Métodos
da tese.

---

## 6. Definição de pronto

O notebook está pronto quando, em um clone limpo do repositório:

1. `data/dados_mock.xlsx` existe e segue a estrutura da seção 2.
2. Abrir `notebook/analise_pesquisa.ipynb` e rodar **Run All** termina sem
   erro.
3. Todos os artefatos da seção 4-Fase I aparecem em `outputs/`.
4. Trocar `CAMINHO_PLANILHA` no Bloco 2 por `data/dados_reais.xlsx` (quando
   existir, com a mesma estrutura) faz a análise rodar contra os dados reais
   sem nenhuma outra alteração de código.
5. Cada bloco do CLAUDE.md (0–14, mais 12.5) tem markdown explicativo,
   preview de entrada, cálculo, preview de saída.
6. Cada item da seção 12 do `spec_artigo_referencia.md` (síntese de rigor)
   está atendido ou explicitamente registrado como follow-up no Bloco 14.

---

## 7. O que **não** está no escopo desta entrega

- Re-execução automática dos geradores (`geradores/construir_notebook.py`
  e `geradores/gerar_dados_mock.py`): existem versionados, mas travados —
  não fazem parte do fluxo de uso normal. Ver `CLAUDE.md`.
- Tabela de MPVs regulatórios concretos (depende de decisão da orientação;
  o Bloco 12.5 fica como *stub* funcional).
- Análises de cegamento inter-observador (kappa/ICC) — depende de dados de
  dupla leitura que não estão na planilha.
- Modelagem confirmatória (regressão, GLM, modelos mistos) — fora do desenho
  do artigo de referência.
- Versionamento dos dados reais no repositório (sensibilidade ética).

---

## 8. Riscos conhecidos

- **Tamanho amostral pequeno**: bootstrap e PCA podem ficar instáveis;
  mitigação = reportar IC e variância explicada com honestidade, não esconder.
- **Excel inconsistente**: a planilha real pode chegar com nomes de colunas
  diferentes; mitigação = Bloco 4 padroniza, Bloco 5 valida cedo e falha alto.
- **Múltiplas comparações**: com 7 marcadores + 9 EPTs = 16 variáveis (120
  pares únicos no Spearman, 16 testes no Mann-Whitney); correção BH é
  obrigatória, não opcional.
- **MPV ausente**: Bloco 12.5 fica visivelmente *stub* até a orientação
  decidir — não inventar limites regulatórios é uma trava de integridade,
  não um defeito do plano.

---

## 9. Próximo passo imediato

`data/dados_mock.xlsx` e `notebook/analise_pesquisa.ipynb` já existem
(gerados via `geradores/`, ver `CLAUDE.md` "Estado atual do repositório").
Próxima tarefa: **revisão humana do notebook** — rodar `Run All`, validar
cada bloco contra a estrutura prevista nas seções 4 e 5, registrar
divergências em `bug_fix/`.
