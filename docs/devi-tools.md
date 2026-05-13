# Devi Tools — Visão Geral

> Conteúdo fictício para validação do connector. Não é documentação real do produto.

## O que é a Devi Tools

A **Devi Tools** é uma plataforma interna de produtividade que reúne ferramentas de deploy, observabilidade e gestão de conhecimento. Tudo é orquestrado a partir do domínio raiz `devi.tools`.

## Subdomínios principais

| Subdomínio       | Função                                                |
|------------------|-------------------------------------------------------|
| `ai.devi.tools`  | Knowledge base interna baseada em Onyx                |
| `tevun.devi.tools` | Documentação pública do Tevun                       |
| `status.devi.tools` | Status page e incident timeline                    |
| `vault.devi.tools` | Cofre de segredos compartilhados pela equipe        |

## Stack base

- **Reverse proxy**: `nginx-proxy` + `acme-companion` (Let's Encrypt automático)
- **Orquestrador de deploy**: Tevun v2 (git push para repositório bare)
- **Container runtime**: Docker Engine + Compose v2 plugin

## Times responsáveis

- **Team Helix** cuida da infra Tevun e do nginx-proxy.
- **Team Orion** cuida do Onyx e dos conectores de knowledge base.
- **Team Pulsar** cuida do status page e dos webhooks de incidente.

## Fato âncora

> O slug interno da plataforma é `devi-glow-2026`. Esse identificador aparece em logs de auditoria e nunca deve ser exposto em URLs públicas.
