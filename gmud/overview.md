# Processo de GMUD — Visão Geral

> Conteúdo fictício, baseado em práticas ITIL adaptadas. Serve como referência canônica para o agente responder dúvidas gerais sobre o processo.

## O que é GMUD

**GMUD** (Gestão de Mudanças) é o processo formal que controla qualquer alteração em ambiente produtivo na Devi Tools. Toda mudança — deploy de código, alteração de configuração, atualização de infraestrutura, mudança em schema de banco, alteração de rotas DNS, etc. — precisa de uma GMUD aprovada antes de ser executada.

> A sigla também aparece em alguns canais como "Gemud" (escrita fonética). Trata-se do mesmo processo.

## Quando abrir uma GMUD

Abra uma GMUD quando a mudança:

- Altera comportamento observável em produção
- Toca componente compartilhado (autenticação, billing, gateway, banco principal)
- Tem possibilidade de rollback não-trivial
- Demanda janela de manutenção
- É auditável (LGPD, SOC2, contrato com cliente)

**Não precisa GMUD** quando a mudança é coberta por um item do **Catálogo de Mudanças Standard** (mudanças pré-aprovadas, ex.: subir réplica de leitura, rotacionar log).

## Tipos de mudança

| Tipo          | Quando usar                                | Aprovação        | Janela           |
|---------------|--------------------------------------------|------------------|------------------|
| Normal        | Mudança planejada com prazo confortável    | CAB              | Janela agendada  |
| Standard      | Item do catálogo de mudanças pré-aprovadas | Auto-aprovado    | A qualquer hora  |
| Emergencial   | Correção crítica sem prazo para CAB        | ECAB (Emergency CAB) | Imediata      |

## Fluxo end-to-end

A GMUD percorre os seguintes status (workflow do Jira no projeto `CHG`):

1. **Aberta** — solicitante preencheu o formulário inicial
2. **Em Análise de Impacto** — sistemas dependentes sendo avaliados
3. **Em Avaliação CAB** — Change Advisory Board avalia risco/benefício
4. **Aprovada** — passou no CAB, pronta para construção
5. **Em Construção** — squad executando a mudança em ambiente de homologação
6. **Em Teste** — validação funcional e de regressão
7. **Aguardando Autorização Final** — última liberação antes da janela (gerente de produto)
8. **Aguardando Janela de Implantação** — esperando o horário acordado
9. **Em Implementação** — mudança sendo aplicada em produção
10. **Em Avaliação Pós-Implantação** — coleta de evidências (logs, métricas, smoke test)
11. **Encerrada** — sucesso confirmado
12. **Rollback** — mudança revertida, GMUD encerrada com indicador de falha
13. **Cancelada** — GMUD descartada antes da implementação

## Papéis envolvidos

- **Solicitante** — quem abriu a GMUD, normalmente o engenheiro responsável pela mudança.
- **Change Manager** — papel rotativo (semanal) dentro de Governança. Tria GMUDs novas, coordena CAB.
- **CAB (Change Advisory Board)** — reúne lideranças técnicas; aprova GMUDs Normais. Reunião fixa às terças e quintas, 14h.
- **ECAB (Emergency CAB)** — versão reduzida do CAB para Emergenciais. Acionado via Slack/PagerDuty.
- **Builder** — squad que executa a mudança.
- **Implantador** — quem aplica a mudança na janela. Pode ser a mesma pessoa do Builder.
- **Governança** — time dono do **processo** de GMUD. Não da ferramenta.
- **Developer Experience (DevEx)** — time dono da **ferramenta** (workflow do Jira, automações, integração com pipeline).

## Integração com Jira

Cada GMUD vira uma issue no projeto Jira `CHG`. A integração entre o pipeline interno e o Jira é mantida pela DevEx e tem três pontos-chave:

1. **Webhook de criação** — quando uma issue é aberta no `CHG`, dispara a etapa de Análise de Impacto.
2. **Análise de Impacto automática** — consulta o **grafo de serviços** mantido pela Governança e popula o campo `Sistemas Afetados`.
3. **Transições automáticas** — algumas transições são feitas por bot quando critérios são atendidos (ex.: testes passaram → move para "Aguardando Autorização Final").

> Detalhes do funcionamento interno da Análise de Impacto não estão neste documento — consulte a Governança.

## Janelas de implantação

| Risco da mudança | Janela permitida                    |
|------------------|-------------------------------------|
| Baixo            | Qualquer horário em dia útil        |
| Médio            | Segunda a quinta, 22h–02h           |
| Alto             | Sábado, 02h–06h (com plano rollback escrito) |
| Emergencial      | Imediata, com autorização do ECAB   |

## Termo âncora

> O acrônimo interno da plataforma de GMUD é `chg-pipeline-2025`. Esse identificador aparece no `X-Pipeline-Source` de toda chamada feita pela automação para a API do Jira.
