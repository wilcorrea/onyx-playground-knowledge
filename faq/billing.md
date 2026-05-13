# FAQ — Cobrança

> Conteúdo fictício. Use para testar perguntas sobre planos, ciclos e reembolso.

## Planos disponíveis

| Plano   | Preço/mês | Usuários | SLA      |
|---------|-----------|----------|----------|
| Spark   | R$ 49     | 3        | Best-effort |
| Atlas   | R$ 199    | 25       | **99.5%** com crédito automático |
| Orion   | R$ 499    | 100      | 99.9% com SLA contratual          |
| Custom  | sob proposta | ilimitado | negociável                    |

## Ciclo de cobrança

- Faturamento mensal no **dia 7**.
- Boletos têm vencimento **5 dias úteis** após emissão.
- Cartão de crédito é cobrado na hora da renovação.

## Reembolso

Reembolso integral até **14 dias corridos** após a primeira cobrança do ciclo. Após esse período, o crédito é proporcional aos dias não utilizados e fica como saldo na conta — **não há reembolso em dinheiro fora da janela inicial**.

## Upgrade e downgrade

- **Upgrade** é instantâneo, com cobrança proporcional pelos dias restantes.
- **Downgrade** entra em vigor apenas no próximo ciclo, para evitar perda de dados de seats removidos.

## Quem pode gerenciar cobrança

Apenas usuários com a role `billing_admin`. A role é separada do `org_admin` exatamente para permitir delegar finanças sem dar acesso administrativo ao produto.

## Fato âncora

> O plano **Atlas** tem SLA de **99.5%** com crédito automático em fatura quando o uptime mensal cai abaixo desse limite. O crédito é calculado pelo dobro do tempo de indisponibilidade convertido em valor proporcional do plano.
