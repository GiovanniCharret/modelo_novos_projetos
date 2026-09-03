# OBJECTIVE

Checklist que ajuda a estruturar o desenvolvimento via arquitetura de projetos com IA para tornar eficazes os outputs.

**Não são listas de atividades para ia**. São para humanos decidirem o que implmentar, dependendo da estrutura do projeto.


## Glossário

a - anulado
f - Revisão Futura
x - concluído
n - Não se aplica
r - Rollback - falhou

## Fases

[x] - Escreva em CLAUDE: Não faça alterações em PROJECT_BULDING.md. As fases e controles de progresso deve ser feitos e registrados em PLAN.md. "> `PROJECT_BUILDING.md` não é alterado (checklist do humano). O controle do projeto do humano vive aqui."
[x] - Construir pasta docs (antiga planning) E criar arquivo PLAN.md
[x] - Escrever no CLAUDE.md que toda a documentação estará em `docs` directory e o key document is PLAN.md
[n] - Criar uma pesquisa do plano equivalente ao texto abaixo:
    - "Realize uma pesquisa abrangente(...) e escreva documentos no diretório de planejamento em XXX_API.md"
    - "Pesquise API. Escreva a documentação com exemplos de código"
    - "Use isso para projetar a API em Python que deve ser usada para XXXX. Documente isso em XXX.md"
    - Por fim, documente a estrutura de código para [OBJETIVO]
[n] - Criar novo arquivo com a estrutura do backend em detalhes, com code snippets mais exemplo, de todas funcionalidades, escreva tudo em XXX_BACKEND.md
[x] - Prepara o Github
[x] - Colar o Ritmo de desenvolvimento
[n] - Regras de segurança em `seguranca_saas_vibecoding.md` respeitadas 
[ ] - Preparar o gitignore
[x] - Crie a pasta "suporte_contexto" (antiga bug_fix)
[x] - Usar Superpowers para planejar as fases e subfases. 
[x] - Adicionar BEHVIORAL_GUIDELINES à pasta do projeto e no claude.
[n] - Após o /init - Leia todo o conteúdo de `docs/`. Depois, escreva o `docs/ADVERSARIAL_REVIEW.md`, que testa as falhas e ambiguidades do script: "Aja como um adversário maximamente competente. Sua tarefa é encontrar todas as ambiguidades, lacunas semânticas e formulações suaves neste documento que permiritiram a você seguir tecnicamente a refra enquanto viala seu espírito. Liste cada brecha com o caminho de exploração específico".
[n] - Solicitar que desenvolvimento do site seja feito em pequenas partes para facilitar o teste humano.
[x] - Mapa de testes (o que teste e como testar) escrito em um arquivo TESTES.md. Explica o teste de cada fase caso queira repetir. "Crie um mapa de testes (o que teste é? como testar?), escrito em um arquivo TESTES.md, que explica o teste de cada fase caso queira repetir. "
[n] - Comparar arquitetura atual e clean Architecture
[n] - Avaliar usar sandbox e WSL2/VSCODE Ubuntu para execução
[n] - Criar e instalar dependências
[x] - Governança de desenvolvimento - **Criar COMPANION_html**, tal como exeplica `DOCS/COMPANION_HTML.md` Que explica os critérios de sucesso de cada fase em `definition of done.md` para humanos poderem acompanhar.
[n] - sinalizar quais arquivos da NÃO SÃO entradas (raiz, scripts antigos, minhas_notas.ignore). É apoio para consulta do humano.
[x] - Criar a pasta `script_antigos` e `src/output` e informar ao claude onde quer que novos e antigos scripts vivam
[x] - Clonar o repositório do thariq https://github.com/ThariqS/html-effectiveness. Crie um html de [tarefa] inspirados nos modelos disponíveis em html-effectveness/ para aumentar minha compreensão das suas atividades e a eficiência das minhas decisões.
[n] - AI Operational assistant prompt - Atue como a personalidade  do documento com o qual estou interagindo. Seu nome é "Assistente de IA". Usando o contexto fornecido, responda à pergunta do usuário da melhor forma possível utilizando os recursos disponibilizados.
[x] - Convenção de documentação do código - "Toda função com docstring explicando, nesta ordem: por que a função existe (o problema que ela resolve / o motivo de ser função separada); a lógica do input ao output, em fases numeradas (Entrada → Fase 1 → Fase 2 → … → Saída), descrevendo o que cada bloco transforma. Os comentários de linha explicam intenção, não o óbvio — alinhado com a regra 8 do CLAUDE.md (corrigido em 2026-08-19: o texto anterior pedia "toda linha comentada, inclusive as óbvias", o que contradizia o CLAUDE.md; o notebook seguiu o CLAUDE.md)."
[n] - Se não houver nada no contexto relevante para a pergunta em questão, apenas diga "Hmm, não tenho certeza" e pare por aí. Recuse-se a responder qualquer pergunta que não seja sobre essas informações. Nunca saia do personagem.
[x] - always include e2e tests to cover important paths. You should always make sure that the plans include a test suite that covers the happy paths and edge cases. Your tests should be high quality and give confidence while covering most of the implementation.
[n] - Reproduzir o site atual como um mock não-funcional (cópia React/Vite, dados, hardcoded) para servir de base/canvas no desenho/design de uma futura nova página. A nova página (que conectará a esta, herdará os estilos e compartilhará o mesmo BD).
[x] - Criar o "claude resume.txt"
[n] - install impeccable
[x] - https://github.com/b-open-io/statusline instalado?
[ ] - Criar a versão de distribuição
[x] - Nunca use siglas ou abreviações técnicas sem escrever o termo completo primeiro. Mais informações estão em `docs/no_technical_abbreviations.md`.
[n] - Deploy da VPS via SSH
[n] - Usará streamlit ou tkinter
 
### Python env
1 - powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
2 - .venv\Scripts\activate
3 - uv venv

### Requirements
4 - uv pip install ....
5 - uv pip freeze > requirements.txt

## Ritmo de desenvolvimento

- Módulos construídos em **partes pequenas**, para o humano acompanhar e testar cada pedaço.
- Critérios de sucesso de cada fase escritos em `docs/definition of done.md` (ainda não existe).
- Todo documento de planejamento novo ganha companion HTML autocontido em `docs/html/`, seguindo o
  prompt de `docs/COMPANION_HTML.md` (referências visuais em `docs/COMPANON_HTML/` — note o nome de
  pasta com typo — e `docs/html-effectiveness/`, hoje vazia).
- Siga `docs/BEHAVIORAL_GUIDELINES.md`: simplicidade primeiro, mudanças cirúrgicas, nada
  especulativo, critérios de verificação explícitos antes de codar.
- Glossário de status nos checklists: `x` concluído · `f` revisão futura · `a` anulado ·
  `n` não se aplica · `r` rollback (falhou) · `[ ]` pendente.
- `suporte_contexto/` — contexto de apoio/bugfix (exemplos de bug difíceis de descrever no prompt);
  hoje vazio.
  - Toda função com docstring explicando, nesta ordem: por que a função existe (o problema que ela resolve / o motivo de ser função separada); a lógica do input ao output, em fases numeradas (Entrada → Fase 1 → Fase 2 → … → Saída), descrevendo o que cada bloco transforma. Os comentários de linha explicam intenção, não o óbvio — alinhado com a regra 8 do CLAUDE.md (corrigido em 2026-08-19: o texto anterior pedia "toda linha comentada, inclusive as óbvias", o que contradizia o CLAUDE.md; o notebook seguiu o CLAUDE.md).