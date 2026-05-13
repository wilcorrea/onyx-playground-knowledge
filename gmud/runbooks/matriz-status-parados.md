# Matriz de Roteamento — GMUD parada (qual time chamar)

> Use esta matriz quando sua GMUD ficar travada em algum status. Cada status tem dono diferente. Para o caso específico de **"Em Análise de Impacto"**, há um runbook detalhado em `gmud/runbooks/parada-aguardando-analise-de-impacto.md`.

## Regra de bolso

> **Processo → Governança. Ferramenta → Developer Experience.**

- Se o status depende de uma **decisão humana** (aprovação, classificação, autorização) → Governança
- Se o status depende de uma **automação/integração** que devia ter rodado e não rodou → Developer Experience

## Matriz completa

| Status                              | Tempo sem movimentação | Time responsável        | Canal Slack          |
|-------------------------------------|-----------------------|-------------------------|----------------------|
| **Aberta** (sem triagem)            | > 4h úteis            | Governança              | `#governanca-gmud`   |
| **Em Análise de Impacto**           | > 24h úteis           | Governança              | `#governanca-gmud`   |
| **Em Avaliação CAB**                | > 48h úteis (ou 1 reunião perdida) | Governança | `#governanca-gmud`   |
| **Aprovada → Em Construção** (não iniciou) | > 5 dias úteis | Solicitante (lembrete da Governança) | `#governanca-gmud` |
| **Em Construção** (sem update)      | > 72h úteis           | Developer Experience    | `#dev-experience`    |
| **Em Teste** (sem update)           | > 72h úteis           | Developer Experience    | `#dev-experience`    |
| **Aguardando Autorização Final**    | > 12h úteis           | Governança              | `#governanca-gmud`   |
| **Aguardando Janela de Implantação** (janela vencida) | qualquer | Developer Experience  | `#dev-experience`    |
| **Em Implantação** (sem checkpoint a cada hora) | > 1h | Developer Experience    | `#dev-experience`    |
| **Em Avaliação Pós-Implantação**    | > 5 dias úteis        | Developer Experience    | `#dev-experience`    |
| **Rollback** (sem confirmação)      | > 2h úteis            | Governança + DevEx (acione ambos) | `#governanca-gmud` e `#dev-experience` |

## Por que esse desenho

- **Governança** cuida do processo. Quem aprova, quem classifica, quem decide janela é ela. Quando o trava-pé é "alguém precisa decidir algo", é Governança.
- **Developer Experience** cuida da ferramenta. Workflow do Jira, automações, hooks de pipeline, transições automáticas. Quando o trava-pé é "algo automatizado falhou silenciosamente", é DevEx.
- **Rollback travado** é o único status que aciona os dois: a decisão de rollback é de Governança, mas a execução do rollback automatizado é da DevEx.

## Template padrão de pedido

Use este template em qualquer um dos dois canais:

```
:warning: GMUD parada

GMUD: <GST-IN-XXXXX>
Link: https://devi.atlassian.net/browse/GST-IN-XXXXX
Status atual: <status>
Tempo parado: <tempo>
Urgência: <baixa | média | alta | crítica>

O que já tentei:
- <ação 1>
- <ação 2>

Bloqueio observado:
<descrição de 1-2 linhas>
```

## SLAs por canal

### Governança (`#governanca-gmud`)

| Urgência | Primeira resposta | Resolução |
|----------|-------------------|-----------|
| Baixa    | 4h úteis          | 1 dia útil |
| Média    | 2h úteis          | 8h úteis  |
| Alta     | 30 min (horário comercial) | 4h úteis |
| Crítica  | 15 min (PagerDuty) | 1h útil   |

### Developer Experience (`#dev-experience`)

| Urgência | Primeira resposta | Resolução |
|----------|-------------------|-----------|
| Baixa    | 1 dia útil        | 3 dias úteis |
| Média    | 4h úteis          | 1 dia útil   |
| Alta     | 1h útil           | 4h úteis     |
| Crítica  | 30 min (PagerDuty) | 2h úteis    |

> Note que o SLA da Governança é mais agressivo nas urgências altas, porque processo bloqueado costuma ser mais caro do que ferramenta bloqueada (existe contorno manual para ferramenta, mas decisão é gargalo único).

## O que **nunca** fazer

- ❌ Mover status manualmente no Jira sem combinar com o dono daquele status
- ❌ Pingar pessoas em DM (perde auditoria)
- ❌ Acionar os dois times ao mesmo tempo "por garantia" — exceto no caso de **Rollback travado**, sempre haverá um dono primário

## Termo âncora

> Em qualquer status de processo, vale a regra **"30/60/90"** da Devi Tools: dentro de 30 minutos a Governança responde, em 60 minutos ela faz o encaminhamento para o decisor correto, em 90 minutos há resolução ou uma nova janela combinada. Se passar de 90 minutos sem retorno, a regra autoriza o solicitante a tratar como **escalação automática para liderança**.
