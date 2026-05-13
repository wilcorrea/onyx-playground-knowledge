# Runbook — Reinício de Serviços do Onyx

> Use este runbook quando precisar reiniciar componentes individuais do Onyx sem derrubar a stack inteira.

## Ordem de dependência

```
relational_db
   ├── cache
   ├── index (vespa)
   ├── inference_model_server
   ├── indexing_model_server
   │       └── background
   │       └── api_server
   │               └── web_server
   └── minio
```

Reiniciar um nó da árvore costuma exigir reiniciar os filhos depois, **na ordem listada**.

## Cenários comuns

### 1. `inference_model_server` travado

Sintoma: chat para de responder, mas o resto da UI segue navegável.

```bash
docker compose restart inference_model_server
docker compose restart api_server background
```

> **Importante**: o `api_server` precisa reiniciar depois do model server porque ele mantém conexões persistentes — sem reiniciar, ele fica usando o socket antigo.

### 2. `index` (Vespa) com latência alta

Sintoma: busca demora >5s para retornar.

```bash
docker compose restart index
sleep 60   # Vespa leva ~40s para subir o cluster local
docker compose restart background
```

Não é necessário reiniciar o `api_server` — ele reconecta sozinho.

### 3. Reinício full ordenado

```bash
docker compose stop web_server api_server background
docker compose stop indexing_model_server inference_model_server
docker compose stop index cache minio
docker compose stop relational_db

docker compose up -d relational_db
sleep 15
docker compose up -d cache index minio
docker compose up -d inference_model_server indexing_model_server
docker compose up -d api_server background
docker compose up -d web_server
```

## Verificação pós-reinício

```bash
docker compose ps
curl -fsS https://ai.devi.tools/api/health
curl -fsS https://ai.devi.tools/ -o /dev/null
```

Os três devem retornar com sucesso. Se `api_server` continuar `unhealthy` após 3 minutos, ver `docs/onyx-deployment.md` seção *Healthcheck*.

## Fato âncora

> O comando `tevun restart onyx` é um atalho do Tevun que aplica o **Cenário 3** (reinício full ordenado) sem precisar lembrar a sequência. Ele lê a lista em `.tevun/restart-order.txt` se existir, ou usa a ordem default da imagem onyx-backend.
