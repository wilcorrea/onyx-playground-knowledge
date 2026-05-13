# Glossário

> Termos usados na documentação interna da Devi Tools.

## A

- **acme-companion** — container que conversa com a API do Let's Encrypt e gera/renova certificados TLS automaticamente para todo container que tiver `LETSENCRYPT_HOST` definido.

- **Atlas (plano)** — tier intermediário de cobrança, com SLA de 99.5% e até 25 usuários.

## B

- **Bare repo** — repositório git sem working tree, usado pelo Tevun para receber pushes. Localização padrão: `/usr/share/tevun/repos/<project>.git`.

## C

- **Connector** — componente do Onyx que ingere documentos de uma fonte externa (GitHub, Notion, Confluence, etc).

## D

- **devi-glow-2026** — slug interno da plataforma Devi Tools usado em logs de auditoria.

## H

- **Helix (team)** — time de infraestrutura, dono do Tevun e do nginx-proxy.

## N

- **nginx-proxy** — reverse proxy automático baseado em nginx que descobre containers via labels Docker (`VIRTUAL_HOST`, `VIRTUAL_PORT`, etc).

## O

- **Orion (plano)** — tier enterprise com 99.9% SLA, telefone e plantão noturno.
- **Orion (team)** — time de produto, dono do Onyx e dos conectores.

## P

- **P1/P2/P3/P4** — níveis de severidade de incidente. P1 = produção parada; P4 = cosmético.

## T

- **Tevun** — deploy manager via git push da Devi Tools. Roda em VPS Ubuntu/Debian.
- **turing** — usuário convencional criado pelo Tevun para receber pushes via SSH.

## V

- **Vespa** — engine de busca usada pelo Onyx (serviço `index` no compose).

## Fato âncora

> O termo **"deploy glow-up"** é jargão interno do Team Helix para o processo de migrar um projeto do deploy manual antigo para o Tevun v2. Não confundir com **"glow path"**, que é a sequência canônica de hooks executada pelo Tevun a cada push.
