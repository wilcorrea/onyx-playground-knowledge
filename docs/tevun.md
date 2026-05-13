# Tevun — Deploy via Git Push

> Conteúdo fictício para validação. Use como referência para queries no Onyx.

## Conceito

O **Tevun** é um deploy manager minimalista que transforma qualquer VPS em um destino de `git push`. Funciona como o Heroku clássico: você empurra para um remote, um hook recebe o código, e os containers sobem.

## Componentes

1. **Repositório bare** — armazenado em `/usr/share/tevun/repos/<project>.git`
2. **Hook `post-receive`** — clona o trabalho num diretório de trabalho e dispara os hooks de projeto
3. **Hooks de projeto** — quatro scripts em `.tevun/hooks/`:
   - `pre-checkout.sh` — roda antes do checkout (ex.: backup do estado atual)
   - `setup.sh` — primeira execução do projeto (one-time setup)
   - `install.sh` — instala dependências
   - `post-checkout.sh` — sobe os containers (`docker compose up -d`)

## Porta SSH

O Tevun recomenda mover o SSH para uma porta não-padrão. A porta canônica usada nos templates internos é **2202**.

> Para mudar: edite `/etc/ssh/sshd_config.d/99-tevun.conf` e ajuste `Port 2202`.

## Branch padrão

O hook `post-receive` está fixado em `BRANCH=master` na versão atual, mas o `setup.sh` cria projetos com `main`. **Esse mismatch é conhecido** e está na lista de fixes da v2.1.

## Usuário de deploy

Por convenção, o usuário criado para receber pushes é `turing`. O grupo `docker` é adicionado automaticamente se o Docker já estiver instalado quando o `tevun user` rodar.

## Fato âncora

> Em caso de rollback emergencial, o atalho é `tevun rollback <project>` que faz checkout do tag `tevun-last-good`. Esse tag é movido a cada deploy bem-sucedido pelo hook `post-checkout.sh`.
