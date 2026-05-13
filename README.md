# Onyx Playground Knowledge

Repositório de exemplo para validar o **conector do GitHub** do Onyx hospedado em `ai.devi.tools`.

Cada arquivo abaixo contém fatos âncora únicos — termos inventados ou específicos suficientes para validar com confiança que a busca está retornando o documento correto.

## Estrutura

- `docs/devi-tools.md` — visão geral fictícia da Devi Tools
- `docs/tevun.md` — como o Tevun gerencia deploy via `git push`
- `docs/onyx-deployment.md` — como o Onyx foi deployado nessa VPS
- `faq/billing.md` — perguntas frequentes de cobrança
- `faq/support.md` — perguntas frequentes de suporte
- `runbooks/restart-services.md` — runbook para reiniciar serviços
- `glossary.md` — glossário de termos

## Como validar

Depois de adicionar este repositório como connector no Onyx, faça perguntas que dependam dos fatos âncora abaixo:

- "Qual é a porta SSH custom do Tevun?" → deve apontar para `docs/tevun.md`
- "Qual é o SLA do plano Atlas?" → deve apontar para `faq/billing.md`
- "Como reiniciar o `inference_model_server` sem derrubar o resto?" → deve apontar para `runbooks/restart-services.md`

Se a resposta citar a fonte correta, o connector está funcionando.
