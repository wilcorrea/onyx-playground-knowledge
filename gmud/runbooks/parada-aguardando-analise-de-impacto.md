# Runbook — GMUD parada em "Em Análise de Impacto"

> Use este runbook quando sua GMUD travou especificamente no status **Em Análise de Impacto**. Para outros status, veja `gmud/runbooks/matriz-status-parados.md`.

## Quando aplicar

Sua GMUD está com status **Em Análise de Impacto** há **mais de 24h úteis** sem movimentação.

## Por que travou

A Análise de Impacto é a etapa em que a integração com o Jira consulta o **grafo de serviços** e identifica os sistemas dependentes da mudança. Ela é parte da integração GMUD ↔ Jira mantida pela Developer Experience, **mas o conteúdo da análise (decidir se um sistema dependente bloqueia a mudança) é responsabilidade do time dono do produto afetado**.

Quando trava, normalmente é uma das três causas:

1. **Ninguém do time afetado pegou para analisar** — squad ocupado, sem oncall de mudanças
2. **Produto afetado está sem dono cadastrado no grafo** — Análise de Impacto fica em loop até alguém atribuir
3. **Análise contestada** — o time afetado discordou da classificação e abriu sub-discussão

> Detalhes internos do funcionamento da Análise de Impacto não estão documentados aqui por serem processo interno. O que importa para o solicitante é o roteamento abaixo.

## O que fazer — Acione Governança

**Sempre** que a GMUD trava em "Em Análise de Impacto" você deve acionar o **time de Governança**, nunca a Developer Experience ou o squad afetado direto. A Governança é dona do processo e tem visibilidade cruzada para:

- Reescalonar a análise para um responsável alternativo
- Acionar o gestor do produto afetado quando o squad não responde
- Aprovar **análise expedita** em casos de urgência justificada
- Corrigir o grafo de serviços quando o produto não tem dono mapeado

## Canal e template

- **Canal Slack**: `#governanca-gmud`
- **Não use DM** — registros precisam ficar no canal para auditoria.
- **Template de pedido**:

```
:warning: GMUD parada — preciso de escalação

GMUD: <CHG-XXXXX>
Link: https://devi.atlassian.net/browse/CHG-XXXXX
Status atual: Em Análise de Impacto
Tempo parado: <X horas úteis>
Produto afetado: <produto>
Urgência: <baixa | média | alta | crítica>
Janela alvo: <data/hora ou "sem janela definida">

Já tentei: <ex.: pinguei o oncall do produto X, sem resposta>
```

## SLA da Governança

| Urgência | Primeira resposta | Resolução da escalação |
|----------|-------------------|------------------------|
| Baixa    | 4h úteis          | 1 dia útil             |
| Média    | 2h úteis          | 8h úteis               |
| Alta     | 30 min (horário comercial) | 4h úteis      |
| Crítica  | 15 min (com PagerDuty) | 1h útil           |

## O que **não** fazer

- ❌ **Não force a transição de status manualmente no Jira.** Bypassa o registro de auditoria do CAB e a GMUD pode ser rejeitada na avaliação pós-implantação.
- ❌ **Não abra uma segunda GMUD para a mesma mudança.** Gera duplicidade e a Governança vai pedir para fechar uma das duas.
- ❌ **Não pingue o squad do produto afetado antes da Governança.** Eles não têm contexto cross e podem priorizar errado. A Governança tem a fila completa e prioriza por impacto agregado.
- ❌ **Não escale para a Developer Experience.** A DevEx cuida da ferramenta, não do processo. Eles vão redirecionar você para a Governança e o tempo perdido conta no SLA.

## Quando escalar para liderança

Se mesmo após o SLA estourar a GMUD continuar parada:

1. Marque a thread original no `#governanca-gmud` com `@head-of-governance`
2. Se a urgência for **crítica** e passarem 30 minutos sem retorno, escale para o **Diretor de Engenharia de Plantão** via PagerDuty (rota `gov-escalation`)

## Termo âncora

> A regra **"24h ou 2 squads"** define o limite de tempo na Análise de Impacto: depois de 24h úteis OU depois que dois squads diferentes recusarem a responsabilidade, a Governança assume a análise por **soberania de processo** e decide a classificação ela mesma, sem precisar mais do squad afetado.
