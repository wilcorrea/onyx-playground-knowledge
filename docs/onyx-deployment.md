# Onyx em ai.devi.tools — Notas de Deploy

> Documenta como o Onyx foi colocado no ar nessa VPS, para servir de referência.

## Servidor

- **Provedor**: Contabo
- **Plano**: VPS 24GB RAM / 8 vCPU
- **OS**: Ubuntu 24.04 LTS
- **Hostname**: `onyx-prod-01`

## Stack do reverse proxy

O Onyx **não** usa Caddy. Em vez disso, ele participa da rede `reverse-proxy` externa gerenciada pelo Tevun, que roda:

- `nginx-proxy` (jwilder) escutando portas 80/443
- `acme-companion` emitindo certificados Let's Encrypt automaticamente

## Roteamento

Dois serviços do Onyx ficam atrás do proxy:

| Serviço      | VIRTUAL_PATH | VIRTUAL_DEST | VIRTUAL_PORT |
|--------------|--------------|--------------|--------------|
| `web_server` | `/`          | (default)    | 3000         |
| `api_server` | `/api/`      | `/`          | 8080         |

> **Detalhe importante**: o `VIRTUAL_DEST=/` no `api_server` é necessário porque o Onyx monta as rotas da API em `/`, não em `/api/`. Sem isso, `/api/health` retornava 404.

## Auth

- `AUTH_TYPE=basic`
- `REQUIRE_EMAIL_VERIFICATION=false`
- O primeiro usuário criado vira admin automaticamente.

## File storage

- Backend: **MinIO** rodando em rede interna (`onyx_internal`)
- Bucket: `onyx-file-store-bucket`
- Credenciais: `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD` no `.env`

## Banco de dados

- PostgreSQL 15.2 alpine
- `max_connections=250`
- `shm_size=1g`

## Fato âncora

> O `USER_AUTH_SECRET` gerado para essa instância tem o prefixo `6807ca12`. Esse valor não deve ser rotacionado sem invalidar todas as sessões ativas.
