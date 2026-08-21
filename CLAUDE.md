# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> `PROJECT_BUILDING.md` não é alterado (checklist do humano). O controle do projeto do humano vive aqui.
> toda a documentação estará em `planning` directory e o key document is PLAN.md

> **NUNCA fazer `git push`** — o repositório permanece local; publicar no GitHub é decisão do humano. Commits locais frequentes, mensagens `feat:`/`fix:`/`test:`/`docs:`/`chore:` em português.

## O que é este repositório (leia primeiro)


## Arquitetura: o pipeline e os contratos entre módulos

Layout achatado (padrão do canônico): módulos em `src/`, um único executável, `.bat` na raiz.
Cada seta abaixo é um **contrato de dataframe** — mudar uma coluna quebra o módulo seguinte.

## Governança e documentação

- `planning/PLAN.md` — **documento-chave**: macrofases F0–F7, decisões e pendências. Registre progresso aqui.
- `planning/DESIGN.md` (D1–D8) · `planning/MODELO_CUSTO.md` (fórmula e fontes) ·
  `planning/PLANO_IMPLEMENTACAO.md` (plano passo a passo com código de cada task) ·
  `planning/definition of done.md` (critério de aceite por fase, para o humano acompanhar).
- Cada documento de planejamento novo ganha companion HTML autocontido em `planning/html/`
  (D4, inspirado em `planning/html-effectiveness/`). Existem hoje: `DESIGN.html`,
  `MODELO_CUSTO.html`, `PLANO_IMPLEMENTACAO.html` — `PLAN.md` ainda não tem companion.
- `planning/PROJECT_BUILDING.md` — checklist do humano, **somente leitura**.
- Glossário de status: `x` concluído · `f` revisão futura · `a` anulado · `n` não se aplica ·
  `r` rollback (falhou) · `[ ]` pendente.
- `suporte_contexto/` — contexto de apoio/bugfix; **hoje vazio**. Ainda não escritos:
  `planning/ADVERSARIAL_REVIEW.md` (D8), `planning/TESTES.md`.

`minhas_notas/` é material de **pesquisa**, nunca entrada de execução:

## Code development pace

- O desenvolvimentos dos módulos deve ser feito em pequenas partes para facilitar o acompanhamento e entendimento humano.
- Explica critérios de sucesso de cada fase em `definition of done.md` para humanos poderem acompanhar.

## Documentation

- Toda função com docstring explicando, nesta ordem: por que a função existe (o problema que ela resolve / o motivo de ser função separada); a lógica do input ao output, em fases numeradas (Entrada → Fase 1 → Fase 2 → … → Saída), descrevendo o que cada bloco transforma. Além disso, toda linha de código comentada — inclusive as que parecem óbvias.

## Convenções herdadas do sistema canônico

- Código, docstrings e comentários em **português sem acento** (evita quebra de encoding).
  Nomes de arquivo de insumo têm acento — use `Path`/raw strings e cuidado com o console cp1252.
- **Formato BR** em CSV/TXT: `sep=";"`, `decimal=","`, `encoding="latin-1"`.
- **Determinismo**: mesma entrada → mesma saída (rota gulosa desempata pelo menor índice,
  `groupby(sort=False)` preserva ordem). Nada de aleatoriedade não semeada.
- **Limitações são explícitas, nunca silenciosas**: `EntradaInvalida` com mensagem pronta para o
  usuário final (erro de dados → exit 1 sem traceback); `print("AVISO: ...")` para descartes e
  fallbacks. Bug de programa continua levantando traceback normal.
- Caminhos relativos à raiz (`RAIZ = Path(__file__).resolve().parent.parent` a partir de `src/`),
  para o `.bat` funcionar de qualquer diretório.
