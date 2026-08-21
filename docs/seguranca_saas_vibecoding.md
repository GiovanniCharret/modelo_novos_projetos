# Guia de Auditoria e Refatoração de Segurança para SaaS (Vibe Coding / Agentic AI)

Este documento foi projetado para ser fornecido diretamente a uma **LLM / Agente de Código** (como Claude Code, Cursor, Copilot, Devins, etc.) para auditar, refatorar e blindar a arquitetura e o código de aplicações SaaS desenvolvidas com assistentes de IA.

---

## 🎯 Instruções Principais para a LLM

Ao analisar a base de código do projeto, execute uma auditoria minuciosa e refatore os trechos afetados seguindo rigorosamente as diretrizes abaixo.

---

## 🛑 As 5 Falhas Críticas de Segurança & Regras de Correção

### 1. Banco de Dados sem Trava de Acesso (RLS Desativado)
* **O Problema:** Ferramentas como Supabase ou Firebase conectam o banco diretamente ao front-end. Sem **Row Level Security (RLS)** ativo, o banco expõe todas as tabelas e dados publicamente a qualquer cliente conectado.
* **Ação Obrigatória da LLM:**
  * Mapear todas as tabelas, coleções e schemas do banco de dados.
  * Garantir que a política de RLS esteja **ativada** em 100% das tabelas.
  * Escrever políticas explícitas garantindo que um usuário autenticado só consiga ler, atualizar, inserir ou deletar registros que pertençam exclusivamente ao seu próprio `user_id`.
  * Sinalizar qualquer tabela pública ou permissão de leitura/escrita aberta (`anon` / `public`).

---

### 2. Validação de Autorização / Admin no Front-end
* **O Problema:** Delegar checagem de permissões (ex: `is_admin`, `role`, planos de assinatura) para o front-end (como no `localStorage`, cookies não seguros ou estados em React/Vue). Qualquer atacante pode alterar a variável no navegador (via Inspecionar Elemento) e ganhar acesso administrativo.
* **Ação Obrigatória da LLM:**
  * Remover qualquer lógica no front-end que decida autorização ou privilégios de rotas/conteúdos.
  * Transferir 100% do controle de permissões, perfis e roles de usuário para o **back-end** ou para **Edge Functions / Serverless Endpoints**.
  * Garantir que todas as APIs e chamadas do sistema verifiquem o JWT / Token de Sessão assinado e validem os privilégios no servidor a cada requisição.

---

### 3. Falha de IDOR (Insecure Direct Object References)
* **O Problema:** Rotas e endpoints recebem IDs sequenciais ou diretos (ex: `/api/pedidos/105`). Sem validação de posse, trocar o ID para `106` ou `107` expõe dados de outros clientes.
* **Ação Obrigatória da LLM:**
  * Revisar todas as rotas e funções de busca de dados que recebam identificadores (`id`, `uuid`, `slug`).
  * Adicionar uma cláusula obrigatória de verificação de propriedade no back-end:
    ```sql
    -- Exemplo de conceito de verificação
    WHERE id = :request_id AND user_id = :authenticated_user_id
    ```
  * Substituir IDs sequenciais numéricos (1, 2, 3...) por **UUIDs v4** para dificultar a enumeração por script kids e rastreadores automáticos.
  * Implementar suporte a **Rate Limiting** nos endpoints públicos para bloquear varredura massiva de requisições.

---

### 4. Segredos e Chaves de API no Front-end (Hardcoded Keys)
* **O Problema:** Gravar chaves privadas de gateways de pagamento, tokens de API (OpenAI, Anthropic, AWS, etc.) diretamente no código-fonte do front-end. O build do front-end injeta tudo em arquivos JavaScript públicos.
* **Ação Obrigatória da LLM:**
  * Varrer todos os arquivos do projeto procurando por chaves, segredos ou tokens gravados diretamente no código (hardcoded).
  * Mover todos os segredos para variáveis de ambiente (`.env`) e garantir que sejam consumidas **exclusivamente do lado do servidor / back-end**.
  * Certificar-se de que o arquivo `.env` e arquivos contendo credenciais estejam presentes no `.gitignore`.
  * Se houver chaves privadas já commitadas no histórico do Git, alertar para revogação e rotação imediata das credenciais.

---

### 5. Input Não Sanitizado e Vulnerabilidade a XSS (Cross-Site Scripting)
* **O Problema:** Confiar cegamente no que o usuário digita nos campos de entrada (formulários, perfis, upload). Inserção de scripts maliciosos (HTML/JS) que executam no navegador de outros usuários ou administradores, permitindo roubo de sessão.
* **Ação Obrigatória da LLM:**
  * Tratar qualquer dado vindo do usuário como **hostil por padrão**.
  * Sanitizar e validar todos os inputs e parâmetros recebidos no back-end usando bibliotecas de validação de schema (ex: `zod`, `pydantic`).
  * Escapar caracteres especiais e proibir renderização de HTML/JS não tratado no front-end (`dangerouslySetInnerHTML` ou equivalentes devem ser auditados e removidos).
  * Validar tipos e extensões de arquivos enviados via rotas de upload.

---

## 🛠️ Ferramentas Open Source e Auditoria Recomendadas

Ao finalizar a refatoração via IA, utilize as seguintes ferramentas gratuitas da comunidade para validação adicional:

1. **OWASP ZAP:** Scanner dinâmico de segurança de aplicações web para simulação de ataques e busca por portas abertas e rotas desprotegidas.
2. **Gitleaks:** Scanner estático para detecção de chaves, segredos e senhas vazadas no histórico do repositório Git.
3. **Semgrep:** Análise estática de código-fonte (SAST) para identificar falhas de segurança em múltiplas linguagens.
4. **Bandit:** Analisador estático focado na detecção de vulnerabilidades e maus usos de segurança em projetos Python.

---

## 📋 Checklist de Auditoria para a LLM Executar

Forneça o feedback da auditoria em formato de tabela com os seguintes tópicos:

- [ ] **RLS e Banco:** Todas as tabelas possuem Row Level Security ativo e restritivo ao `user_id`?
- [ ] **Autorização no Server:** O front-end é usado apenas para renderização e o controle de papéis/admin está 100% no servidor?
- [ ] **IDOR / Propriedade:** Todas as rotas de busca garantem que o registro pertence ao usuário logado?
- [ ] **Variáveis de Ambiente:** Nenhuma chave privada ou segredo está exposto no front-end ou commitado no repositório?
- [ ] **Sanitização de Inputs:** Todas as entradas de dados possuem validação estrita no servidor e proteção contra XSS?
