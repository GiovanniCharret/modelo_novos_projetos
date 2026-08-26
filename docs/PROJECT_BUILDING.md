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

[ ] - Escreva em CLAUDE: Não faça alterações em PROJECT_BULDING.md. As fases e controles de progresso deve ser feitos e registrados em PLAN.md. "> `PROJECT_BUILDING.md` não é alterado (checklist do humano). O controle do projeto do humano vive aqui."
[ ] - Construir pasta docs (antiga planning) E criar arquivo PLAN.md
[ ] - Escrever no CLAUDE.md que toda a documentação estará em `docs` directory e o key document is PLAN.md
[ ] - Criar uma pesquisa do plano equivalente ao texto abaixo:
    - "Realize uma pesquisa abrangente(...) e escreva documentos no diretório de planejamento em XXX_API.md"
    - "Pesquise API. Escreva a documentação com exemplos de código"
    - "Use isso para projetar a API em Python que deve ser usada para XXXX. Documente isso em XXX.md"
    - Por fim, documente a estrutura de código para [OBJETIVO]
[ ] - Criar novo arquivo com a estrutura do backend em detalhes, com code snippets mais exemplo, de todas funcionalidades, escreva tudo em XXX_BACKEND.md
[ ] - Prepara o Github
[ ] - Regras de segurança em `seguranca_saas_vibecoding.md` respeitadas 
[ ] - Preparar o gitignore
[ ] - Crie a pasta "suporte_contexto" (antiga bug_fix)
[ ] - Usar Superpowers para planejar as fases e subfases. 
[ ] - Adicionar BEHVIORAL_GUIDELINES à pasta do projeto e no claude.
[ ] - Após o /init - Leia todo o conteúdo de `docs/`. Depois, escreva o `docs/ADVERSARIAL_REVIEW.md`, que testa as falhas e ambiguidades do script: "Aja como um adversário maximamente competente. Sua tarefa é encontrar todas as ambiguidades, lacunas semânticas e formulações suaves neste documento que permiritiram a você seguir tecnicamente a refra enquanto viala seu espírito. Liste cada brecha com o caminho de exploração específico".
[ ] - Solicitar que desenvolvimento do site seja feito em pequenas partes para facilitar o teste humano.
[ ] - Mapa de testes (o que teste e como testar) escrito em um arquivo TESTES.md. Explica o teste de cada fase caso queira repetir. 
[ ] - Comparar arquitetura atual e clean Architecture
[ ] - Avaliar usar sandbox e WSL2/VSCODE Ubuntu para execução
[ ] - Criar e instalar dependências
[ ] - Governança de desenvolvimento - **Criar o html** Que explica os critérios de sucesso de cada fase em `definition of done.md` para humanos poderem acompanhar.
[ ] - sinalizar quais arquivos da NÃO SÃO entradas (raiz, scripts antigos, minhas_notas.ignore). É apoio para consulta do humano.
[ ] - Criar a pasta `script_antigos` e `src/output` e informar ao claude onde quer que novos e antigos scripts vivam
[ ] - Clonar o repositório do thariq https://github.com/ThariqS/html-effectiveness. Crie um html de [tarefa] inspirados nos modelos disponíveis em html-effectveness/ para aumentar minha compreensão das suas atividades e a eficiência das minhas decisões.
[ ] - AI Operational assistant prompt - Atue como a personalidade  do documento com o qual estou interagindo. Seu nome é "Assistente de IA". Usando o contexto fornecido, responda à pergunta do usuário da melhor forma possível utilizando os recursos disponibilizados.
[ ] - Convenção de documentação do código - "Toda função com docstring explicando, nesta ordem: por que a função existe (o problema que ela resolve / o motivo de ser função separada); a lógica do input ao output, em fases numeradas (Entrada → Fase 1 → Fase 2 → … → Saída), descrevendo o que cada bloco transforma. Os comentários de linha explicam intenção, não o óbvio — alinhado com a regra 8 do CLAUDE.md (corrigido em 2026-08-19: o texto anterior pedia "toda linha comentada, inclusive as óbvias", o que contradizia o CLAUDE.md; o notebook seguiu o CLAUDE.md)."
[ ] - Se não houver nada no contexto relevante para a pergunta em questão, apenas diga "Hmm, não tenho certeza" e pare por aí. Recuse-se a responder qualquer pergunta que não seja sobre essas informações. Nunca saia do personagem.
[ ] - always include e2e tests to cover important paths. You should always make sure that the plans include a test suite that covers the happy paths and edge cases. Your tests should be high quality and give confidence while covering most of the implementation.
[ ] - Reproduzir o site atual como um mock não-funcional (cópia React/Vite, dados, hardcoded) para servir de base/canvas no desenho/design de uma futura nova página. A nova página (que conectará a esta, herdará os estilos e compartilhará o mesmo BD).
[ ] - Criar o "claude resume.txt"
[ ] - install impeccable
[ ] - https://github.com/b-open-io/statusline instalado?
[ ] - Leitura do ecc_dashboard.py
[ ] - Criar a versão de distribuição
[ ] - No techinical abbreviations
 
### Python env
1 - powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
2 - .venv\Scripts\activate
3 - uv venv

### Requirements
4 - uv pip install ....
5 - uv pip freeze > requirements.txt