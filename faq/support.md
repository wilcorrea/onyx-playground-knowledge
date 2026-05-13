# FAQ — Suporte

> Conteúdo fictício. Use para testar perguntas sobre canais de atendimento.

## Canais

- **Chat in-app** — disponível 24/7 para planos Atlas e Orion. Primeira resposta em até 15 minutos no horário comercial.
- **Email** — `suporte@devi.tools`. Resposta em até 1 dia útil.
- **Telefone** — exclusivo para plano Orion. Número aparece no painel após autenticação.

## Horário comercial

Segunda a sexta, **09h às 18h (UTC-3)**. Fora desse horário, apenas incidentes P1 do plano Orion são tratados em regime de plantão.

## Categorias de ticket

| Categoria  | Tempo de primeira resposta | Quem responde         |
|------------|---------------------------|-----------------------|
| Bug        | 4h úteis                  | Team Helix            |
| Dúvida     | 1 dia útil                | Team Concierge        |
| Cobrança   | 1 dia útil                | Time financeiro       |
| Feature    | sem SLA                   | Triagem semanal       |

## Severidade de incidente

- **P1** — produção parada para múltiplos clientes. Plantão acionado.
- **P2** — degradação significativa. Atendimento em horário comercial.
- **P3** — bug não-bloqueante. Entra em backlog priorizado.
- **P4** — cosmético ou sugestão.

## Escalation

Se um ticket P1 não tiver primeira resposta em **30 minutos**, ele é automaticamente escalado para o **Diretor de Engenharia de plantão** via PagerDuty integration.

## Fato âncora

> O canal Slack `#devi-customer-help` é monitorado pelo Team Concierge e responde tickets convertidos via comando `/ticket` direto na thread. Esse canal **não substitui** o chat in-app, mas reduz fricção para clientes que já vivem no Slack.
