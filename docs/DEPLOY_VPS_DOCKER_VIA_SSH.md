# Deploy de um projeto Docker num VPS, via SSH — runbook genérico

Subir uma aplicação conteinerizada num VPS **zerado**, a partir do Windows/PowerShell,
sem depender do painel do provedor a não ser em emergência.

**Cenário coberto:** o repositório tem um `docker-compose.yml` que sobe um ou mais
containers, sendo **um deles um servidor web publicando a porta 80**. O container fica
preso em `localhost` e um **Nginx no host** faz de porteiro em 80/443, com TLS do
Let's Encrypt. Esse desenho permite trocar a aplicação sem mexer no certificado.

---

## Variáveis — preencha antes de começar

Substitua em todos os comandos:

| Placeholder | Significado | Exemplo |
|---|---|---|
| `<USUARIO>` | usuário SSH do VPS | `root` |
| `<IP>` | IP público do VPS | `203.0.113.10` |
| `<DOMINIO>` | domínio com registro A apontando para `<IP>` | `meuapp.com.br` |
| `<PROJETO>` | nome curto: pasta em `/opt/` e nome do vhost | `meuapp` |
| `<REPO_LOCAL>` | caminho do repositório na sua máquina | `C:\dev\meuapp` |
| `<SERVICO_WEB>` | nome, no compose, do serviço que publica a porta 80 | `frontend` |
| `<PORTA>` | porta local onde o container vai ficar | `8080` |

**Pré-requisitos:** Docker + plugin `compose` instalados no VPS; DNS de `<DOMINIO>`
(e `www`, se for usar) já apontando para `<IP>`; OpenSSH no Windows (nativo desde o 10).

> **Regra de ouro:** rode `ssh` **sozinho** primeiro. Só depois que o prompt virar
> `<USUARIO>@servidor:~#` é que você está no Linux e pode colar os blocos `bash`.
> Colar tudo junto faz o PowerShell tentar interpretar `&&`, que ele não aceita.

---

## Parte 0 — Acesso SSH

Numa conexão SSH há **duas** provas de identidade, em direções opostas. Entender isso
resolve quase todo problema de acesso:

1. **O servidor prova quem é, para você** — a chave de host, gravada em `known_hosts`.
2. **Você prova quem é, para o servidor** — senha ou par de chaves.

### 0.1 Se a impressão digital do servidor mudou

Acontece sempre que o VPS é reinstalado: instalação nova gera chave de host nova. O SSH
recusa a conexão até você resolver o conflito. Antes de apagar o registro antigo,
**confirme que não é ataque**: consulte a chave pelo IP e pelo domínio — se as duas
baterem entre si, é reinstalação; se divergirem, pare e investigue.

```powershell
# Mostra a chave que o servidor apresenta HOJE, consultando pelo IP.
ssh-keyscan -t ed25519 <IP> | ssh-keygen -lf -
# Mesma consulta pelo domínio — as duas impressões digitais TÊM que ser idênticas.
ssh-keyscan -t ed25519 <DOMINIO> | ssh-keygen -lf -
# Só depois de conferir: apaga o registro antigo do domínio.
ssh-keygen -R <DOMINIO>
# Apaga também o registro antigo do IP.
ssh-keygen -R <IP>
```

Na conexão seguinte, responda `yes` para gravar a chave nova.

### 0.2 Autenticar por chave (elimina a senha)

Num VPS recém-instalado o usuário administrativo costuma ficar **sem senha válida**: o
SSH até oferece o método `password`, mas nenhuma senha funciona e a conexão fecha na
primeira tentativa. Autenticação por chave resolve — e é mais segura, porque a chave
privada nunca trafega pela rede.

```powershell
# Cria o par de chaves SÓ se ainda não existir (a privada nunca sai desta máquina).
if (-not (Test-Path ~\.ssh\id_ed25519)) { ssh-keygen -t ed25519 -N '""' -f ~\.ssh\id_ed25519 }
# Exibe a chave PÚBLICA — é esta linha que vai para o servidor. Copie inteira.
type ~\.ssh\id_ed25519.pub
```

**O impasse do ovo e da galinha:** para instalar a chave por SSH seria preciso já
conseguir entrar por SSH. A saída é um **canal fora de banda**: praticamente todo
provedor oferece um console pelo navegador (VNC/serial), ligado direto na máquina
virtual. Ele **não passa pelo `sshd`** — a autenticação já aconteceu quando você entrou
no painel. Use-o exatamente uma vez, para colar:

```bash
# Cria a pasta de configuração do SSH do usuário; o sshd exige permissão restrita nela.
mkdir -p ~/.ssh && chmod 700 ~/.ssh
# Acrescenta sua chave pública à lista de quem pode entrar. COLE a linha do seu .pub.
echo 'ssh-ed25519 AAAA...SUA_CHAVE_PUBLICA... comentario' >> ~/.ssh/authorized_keys
# Sem esta permissão o sshd IGNORA o arquivo em silêncio e continua pedindo senha.
chmod 600 ~/.ssh/authorized_keys
```

Teste na sua máquina — deve entrar **sem pedir nada**:

```powershell
ssh <USUARIO>@<IP>
```

### 0.3 Se ainda falhar, diagnostique antes de chutar

```powershell
# -v mostra quais métodos o servidor aceita e em que ponto a autenticação morre.
ssh -v <USUARIO>@<IP>
```

| O que aparece | O que significa | O que fazer |
|---|---|---|
| `Authentications that can continue: publickey` | senha desabilitada no servidor | instalar a chave (0.2) |
| `Offering public key: …` e nova recusa | a chave não está no `authorized_keys` | repetir 0.2, conferir o `chmod 600` |
| `Permission denied, please try again` | senha errada de fato | redefinir a senha no painel |
| `Too many authentication failures` | o cliente ofereceu chaves demais antes | `ssh -o PubkeyAuthentication=no <USUARIO>@<IP>` |

---

## Parte 1 — Empacotar e enviar (PowerShell)

`git archive` exporta **apenas os arquivos versionados** do commit atual: sem `.git`, sem
`node_modules`, sem ambiente virtual. Vantagem sobre `git clone` no servidor: **nenhuma
credencial do GitHub precisa existir no VPS**, e sobe exatamente o commit que você
conferiu. Vantagem sobre `scp -r` da pasta: não arrasta lixo local.

```powershell
# Entra no repositório que será publicado.
cd <REPO_LOCAL>
# Confirma que não há alteração pendente — o pacote leva o COMMIT, não o working tree.
git status --short
# Registra qual commit está sendo empacotado (anote: é o que ficará no ar).
git log -1 --oneline
# Gera o pacote FORA do repo. Atenção: /tmp NÃO existe no Windows, use caminho Windows.
git archive --format=tar.gz -o ..\deploy.tgz HEAD
# Envia pelo túnel do SSH, com a mesma chave da Parte 0 (não pede senha).
scp ..\deploy.tgz <USUARIO>@<IP>:/tmp/
```

> Se o repositório versiona arquivos de configuração ou segredos (`.env`, credenciais),
> eles vão dentro do pacote — é o que faz a aplicação subir sem configuração manual, e é
> o motivo pelo qual esse repositório precisa continuar privado. Se **não** versiona,
> copie esses arquivos separadamente por `scp` antes de subir os containers.

---

## Parte 2 — Subir os containers

```powershell
# Entre no servidor SOZINHO. Espere o prompt mudar antes de colar o bloco seguinte.
ssh <USUARIO>@<IP>
```

```bash
# Confirma que o Docker e o plugin compose existem antes de qualquer coisa.
docker --version && docker compose version
# Cria o destino, extrai o pacote e apaga o .tgz (pode conter segredos; não deixe sobrando).
mkdir -p /opt/<PROJETO> && tar -xzf /tmp/deploy.tgz -C /opt/<PROJETO> && rm /tmp/deploy.tgz
# Confere que o conteúdo do repositório chegou inteiro.
ls /opt/<PROJETO>
```

O passo seguinte prende o container em `127.0.0.1:<PORTA>` e **libera a porta 80** para o
Nginx do host. Um arquivo `docker-compose.override.yml` faz isso **sem editar o compose
versionado** — o repositório continua idêntico ao original.

```bash
# Vá para a pasta onde está o docker-compose.yml (ajuste se estiver na raiz do repo).
cd /opt/<PROJETO>/docker
# Remove override antigo, se houver — torna este passo repetível sem acumular lixo.
rm -f docker-compose.override.yml
# `!override` SUBSTITUI a lista de portas do compose base; sem ele as duas se somam e a 80
# continuaria publicada, brigando com o Nginx do host.
# printf em vez de heredoc de propósito: um `EOF` indentado NÃO fecha heredoc e trava o shell.
printf 'services:\n  <SERVICO_WEB>:\n    ports: !override\n      - "127.0.0.1:<PORTA>:80"\n' > docker-compose.override.yml
# Confere o YAML gerado antes de usar (indentação de YAML é implacável).
cat docker-compose.override.yml
# Constrói as imagens e sobe em segundo plano. A 1ª vez demora bastante.
docker compose up -d --build
```

Teste **antes** de expor à internet:

```bash
# O serviço web tem que aparecer como 127.0.0.1:<PORTA>->80/tcp.
docker compose ps
# A aplicação responde na porta interna (espera 200).
curl -s -o /dev/null -w 'web: %{http_code}\n' http://127.0.0.1:<PORTA>/
# Se houver endpoint de health, confirme que a API também responde.
curl -s http://127.0.0.1:<PORTA>/api/health; echo
```

Se algo falhar, `docker compose logs --tail 50` mostra o motivo. Nada está público ainda.

---

## Parte 3 — Nginx no host + HTTPS

```bash
# Nginx (porteiro em 80/443) e certbot (certificado gratuito Let's Encrypt).
apt update && apt install -y nginx certbot python3-certbot-nginx
# Vhost repassando TUDO para o container. Aspas SIMPLES no printf: $host e afins ficam
# literais, sem o bash expandir. client_max_body_size: sem isso, upload grande vira 413.
# Inclua o `www` no server_name desde já, ou o certbot deixa o www respondendo 404.
printf 'server {\n  listen 80;\n  server_name <DOMINIO> www.<DOMINIO>;\n  client_max_body_size 50m;\n  location / {\n    proxy_pass http://127.0.0.1:<PORTA>;\n    proxy_set_header Host $host;\n    proxy_set_header X-Real-IP $remote_addr;\n    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;\n    proxy_set_header X-Forwarded-Proto $scheme;\n  }\n}\n' > /etc/nginx/sites-available/<PROJETO>
# Ativa o site criando o link simbólico em sites-enabled.
ln -sf /etc/nginx/sites-available/<PROJETO> /etc/nginx/sites-enabled/<PROJETO>
# Remove o site padrão, senão ele pode responder no lugar do seu.
rm -f /etc/nginx/sites-enabled/default
# Valida a sintaxe ANTES de recarregar — um reload com erro derruba o serviço.
nginx -t && systemctl reload nginx
# Libera as portas no firewall (inofensivo se o ufw estiver desligado).
ufw allow 'Nginx Full' 2>/dev/null; ufw allow OpenSSH 2>/dev/null
# Teste em HTTP antes de pedir o certificado (o certbot precisa da porta 80 funcionando).
curl -s -o /dev/null -w 'http: %{http_code}\n' http://<DOMINIO>/
```

```bash
# Interativo de propósito: o certbot pergunta o e-mail e os termos, então você não
# precisa editar a linha antes de colar. --redirect já configura HTTP -> HTTPS.
certbot --nginx -d <DOMINIO> -d www.<DOMINIO> --redirect
```

Verificação final:

```bash
# Aplicação servida sob TLS (espera 200).
curl -s -o /dev/null -w 'https: %{http_code}\n' https://<DOMINIO>/
# O www também funciona (espera 200).
curl -s -o /dev/null -w 'www:   %{http_code}\n' https://www.<DOMINIO>/
# HTTP redireciona para HTTPS (espera 301).
curl -s -o /dev/null -w 'redir: %{http_code}\n' http://<DOMINIO>/
```

---

## Atualizar depois de novos commits

```powershell
# Regenera o pacote a partir do commit novo.
cd <REPO_LOCAL>
git archive --format=tar.gz -o ..\deploy.tgz HEAD
# Envia.
scp ..\deploy.tgz <USUARIO>@<IP>:/tmp/
```

```bash
# Extrai POR CIMA (sobrescreve os arquivos versionados) e limpa o pacote.
tar -xzf /tmp/deploy.tgz -C /opt/<PROJETO> && rm /tmp/deploy.tgz
# Reconstrói e sobe. O Nginx e o certificado do host não precisam de nada.
cd /opt/<PROJETO>/docker && docker compose up -d --build
```

> `tar -xzf` **sobrescreve**, mas não apaga: arquivo removido no repositório continua no
> servidor. Se isso importar, extraia numa pasta nova e troque, em vez de sobrepor.

---

## Operação e diagnóstico

```bash
# Logs em tempo real de um serviço (Ctrl+C para sair).
docker compose logs <SERVICO> -f
# Reinicia um serviço — necessário quando a app lê um arquivo só na inicialização.
docker compose restart <SERVICO>
# Abre um shell dentro do container, para inspecionar o que realmente foi para a imagem.
docker compose exec <SERVICO> sh
# Derruba tudo (o Nginx do host segue de pé e passará a responder 502).
docker compose down
# Espaço em disco: builds repetidos acumulam imagens órfãs e enchem o VPS.
docker system df
# Limpa imagens e caches não usados (confirma antes de executar).
docker system prune -a
# Renovação do certificado é automática; confira se o timer está ativo.
systemctl list-timers | grep certbot
```

---

## Armadilhas comuns

| Sintoma | Causa | Correção |
|---|---|---|
| `O token '&&' não é um separador válido` | bloco `bash` colado no PowerShell | rode `ssh` sozinho; espere o prompt mudar |
| `could not open '/tmp/…' for writing` | `/tmp` não existe no Windows | use caminho Windows (`..\deploy.tgz`) |
| Shell trava após `cat > arq <<'EOF'` | `EOF` indentado não fecha heredoc | `Ctrl+C` e refaça com `printf` |
| `<palavra>: command not found` | texto explicativo colado junto | cole só o conteúdo dos blocos de código |
| `Connection closed` após 1 tentativa de senha | usuário sem senha válida | autentique por chave (0.2) |
| Impressão digital diferente da gravada | VPS reinstalado (chave de host nova) | confira IP × domínio, depois `ssh-keygen -R` |
| Upload grande responde 413 | falta `client_max_body_size` | já incluído no vhost da Parte 3 |
| `502 Bad Gateway` | container caiu ou mudou de porta | `docker compose ps` e `logs` |
| certbot falha ao validar | DNS não aponta para o VPS, ou porta 80 fechada | confira o registro A e o firewall |
| Porta 80 ocupada ao subir o compose | override ausente ou sem `!override` | refaça o override e `docker compose up -d` |

---

## Limitações deste desenho — decida conscientemente

- **Arquivos copiados na imagem não são atualizáveis por fora.** Se o `Dockerfile` faz
  `COPY . .`, editar ou enviar por `scp` um arquivo em `/opt/<PROJETO>/` **não tem
  efeito**: o container continua com o conteúdo do build. Só um `--build` novo atualiza.
- **Dados gravados pela aplicação somem no próximo `--build`** — qualquer arquivo que o
  código escreva vive dentro do container.

As duas se resolvem com volumes no override, quando fizer sentido:

```yaml
services:
  <SERVICO>:
    volumes:
      # Pasta de dados atualizada por fora, sem rebuild.
      - ../dados:/app/dados
      # Arquivo que a aplicação grava e precisa sobreviver ao rebuild.
      - ../estado.json:/app/estado.json
```

- **Sem `git pull` no servidor**, por opção: não há credencial lá. Toda atualização é
  `git archive` + `scp`. Se um dia precisar de pull, gere um token com escopo de leitura
  ou instale uma deploy key.
- **Um único host, sem redundância.** Reinstalar o VPS apaga tudo; o que garante a volta é
  o repositório mais este runbook — e um backup dos volumes, se você criar algum.
