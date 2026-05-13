# Processo de GMUD — Visão Geral

> Conteúdo fictício, baseado em práticas ITIL adaptadas. Serve como referência canônica para o agente responder dúvidas
> gerais sobre o processo.

## O que é GMUD

**GMUD** (Gestão de Mudanças) é o processo formal que controla qualquer alteração em ambiente produtivo na Devi Tools.
Toda mudança — deploy de código, alteração de configuração, atualização de infraestrutura, mudança em schema de banco,
alteração de rotas DNS, etc. — precisa de uma GMUD aprovada antes de ser executada.

> A sigla também aparece em alguns canais como "Gemud" (escrita fonética). Trata-se do mesmo processo.

## Quando abrir uma GMUD

Abra uma GMUD quando a mudança:

- Altera comportamento observável em produção
- Toca componente compartilhado (autenticação, billing, gateway, banco principal)
- Tem possibilidade de rollback não-trivial
- Demanda janela de manutenção
- É auditável (LGPD, SOC2, contrato com cliente)

**Não precisa GMUD** quando a mudança é coberta por um item do **Catálogo de Mudanças Standard** (mudanças
pré-aprovadas, ex.: subir réplica de leitura, rotacionar log).

## Modos de criação: Manual e Automática

Toda GMUD nasce de um dos dois modos abaixo. O **fluxo principal de aprovação é o mesmo** nos dois casos — o que muda é
quem cria a issue e como os metadados são preenchidos.

### Manual

O solicitante abre uma issue diretamente no projeto `CHG` do Jira, preenchendo o formulário de mudança. Usado para
mudanças que **não envolvem código**: configuração de infra, alteração de DNS, escalonamento de banco, mudanças de
processo, manutenção física, etc.

### Automática

Geradas por **pipelines de CI configuradas via webhook nos repositórios**. Toda vez que um Pull Request é aberto num
repositório com integração GMUD ativa, o pipeline:

1. Cria uma issue no projeto `CHG` automaticamente
2. Associa a GMUD ao Pull Request (campo `pull_request_url` na issue + status check `gmud/approved` no PR)
3. Coleta metadados a partir do PR (autor, descrição, arquivos tocados, plano de rollback se houver)
4. Submete a GMUD ao **mesmo fluxo end-to-end** das manuais

**Diferentes pipelines podem criar GMUDs automáticas.** Cada repositório pode ter sua própria configuração de coleta
(arquivo `.gmud.yml` na raiz), mas o fluxo principal de status e aprovações é idêntico — só a etapa de extração de
metadados varia entre pipelines.

> Antes de abrir um PR para produção, confirme se o repositório tem integração GMUD ativada. Se não tiver, abra uma
> GMUD manual ANTES do merge — caso contrário, a release ficará bloqueada.

## Tipos de mudança

| Tipo        | Quando usar                                | Aprovação            | Janela          |
|-------------|--------------------------------------------|----------------------|-----------------|
| Normal      | Mudança planejada com prazo confortável    | CAB                  | Janela agendada |
| Standard    | Item do catálogo de mudanças pré-aprovadas | Auto-aprovado        | A qualquer hora |
| Emergencial | Correção crítica sem prazo para CAB        | ECAB (Emergency CAB) | Imediata        |

## Fluxo end-to-end

A GMUD percorre os seguintes status (workflow do Jira no projeto `CHG`):

1. **Aberta** — solicitante preencheu o formulário inicial
2. **Em Análise de Impacto** — sistemas dependentes sendo avaliados
3. **Em Avaliação CAB** — Change Advisory Board avalia risco/benefício
4. **Aprovada** — passou no CAB, pronta para construção
   - **Aprovação Janela de Bloqueio** *(branch alternativo, durante code freeze)* — quando uma janela de bloqueio é
     iniciada, GMUDs que estavam em "Aprovada" transitam para este sub-status. Nele é possível disparar um deploy
     emergencial reutilizando a GMUD existente, **sem criar uma nova release**. Detalhes na seção *Janela de Bloqueio*.
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
- **CAB (Change Advisory Board)** — reúne lideranças técnicas; aprova GMUDs Normais. Reunião fixa às terças e quintas,
  14h.
- **ECAB (Emergency CAB)** — versão reduzida do CAB para Emergenciais. Acionado via Slack/PagerDuty.
- **Builder** — squad que executa a mudança.
- **Implantador** — quem aplica a mudança na janela. Pode ser a mesma pessoa do Builder.
- **Governança** — time dono do **processo** de GMUD. Não da ferramenta.
- **Developer Experience (DevEx)** — time dono da **ferramenta** (workflow do Jira, automações, integração com
  pipeline).

## GMUD e Pull Requests

GMUDs **automáticas** ficam bidirecionalmente vinculadas a um Pull Request. Esse vínculo cria três regras de ouro que
todo solicitante precisa entender:

### 1. Merge do PR é bloqueado até a GMUD ser aprovada

O pipeline aplica um **status check** chamado `gmud/approved` em todo PR vinculado a uma GMUD. Esse check só passa
quando a GMUD chega ao status **Aprovada** (ou superior). Antes disso, o botão de merge fica desabilitado **mesmo que
todos os reviewers tenham aprovado o PR**.

> Em outras palavras: review do PR e aprovação da GMUD são duas validações independentes, e o merge exige as duas.

### 2. Fechar PR sem merge cancela a GMUD automaticamente

Se o PR for **fechado sem ser mesclado** (closed, not merged), o webhook `pull_request.closed` dispara um cancelamento
automático da GMUD associada. Aparece o motivo `auto-cancelled: PR closed without merge` nos comentários da issue do
`CHG`.

O cancelamento manual também é possível a qualquer momento — basta mover a issue para **Cancelada** no Jira. O
automatismo só cobre o caminho mais comum (PR descartado).

### 3. Release só pode ser gerada com PR aprovado **E** GMUD aprovada

Em projetos que usam **releases como mecanismo de delivery do artefato final** (não fazem deploy a partir do branch),
a release só é cortada quando os dois critérios estão atendidos:

- PR aprovado por reviewers + status checks passando (incluindo `gmud/approved`)
- GMUD associada em status **Aprovada** (ou um dos status descendentes válidos)

Se faltar qualquer um, a tentativa de criar a release falha com `release_blocked_by_gmud` no pipeline.

## Integração com Jira

Cada GMUD vira uma issue no projeto Jira `CHG`. A integração entre o pipeline interno e o Jira é mantida pela DevEx e
tem três pontos-chave:

1. **Webhook de criação** — quando uma issue é aberta no `CHG`, dispara a etapa de Análise de Impacto.
2. **Análise de Impacto automática** — consulta o **grafo de serviços** mantido pela Governança e popula o campo
   `Sistemas Afetados`.
3. **Transições automáticas** — algumas transições são feitas por bot quando critérios são atendidos (ex.: testes
   passaram → move para "Aguardando Autorização Final").

> Detalhes do funcionamento interno da Análise de Impacto não estão neste documento — consulte a Governança.

## Janelas de implantação

| Risco da mudança | Janela permitida                             |
|------------------|----------------------------------------------|
| Baixo            | Qualquer horário em dia útil                 |
| Médio            | Segunda a quinta, 22h–02h                    |
| Alto             | Sábado, 02h–06h (com plano rollback escrito) |
| Emergencial      | Imediata, com autorização do ECAB            |

## Janela de Bloqueio (Code Freeze)

**Não confundir com "Janela de implantação"**: a *janela de implantação* é o horário **permitido** para deploy de uma
GMUD específica; a **janela de bloqueio** é um período em que deploys regulares ficam **suspensos** por motivos
calendarizados (véspera de feriado, lançamento grande, fim de mês contábil, auditoria SOC2, evento de marketing, etc).

### O que muda quando uma janela de bloqueio está ativa

- **GMUDs novas** só podem ser do tipo **Emergencial** (Normal e Standard ficam suspensas)
- **GMUDs já aprovadas** transitam automaticamente para o sub-status **Aprovação Janela de Bloqueio**
- **Geração de releases automáticas fica suspensa** — pipelines não cortam tag durante o freeze
- **Deploys precisam reutilizar uma GMUD já aprovada** — não há caminho automático de "criar release nova"

### Como funciona o deploy durante o freeze

Esse é o ponto-chave: durante a janela de bloqueio, uma GMUD em status **Aprovação Janela de Bloqueio** pode ser usada
como **gatilho manual de deploy emergencial**. O deploy:

1. **Não cria uma release nova** — aponta para o último artefato já gerado antes do freeze
2. **Reutiliza a GMUD associada** que já passou pelo CAB
3. **Requer aprovação on-the-fly do Change Manager de plantão da semana** — não passa pelo CAB de novo (já passou),
   mas o Change Manager precisa confirmar que a emergência justifica o bypass do freeze
4. **Gera entrada no log de exceções de freeze** — auditoria reforçada, todo deploy nessa modalidade fica registrado

Em uma frase: durante uma janela de bloqueio, GMUDs aprovadas viram **"GMUDs de gatilho manual"** — ficam à disposição
para emergências, mas perdem o automatismo de release.

### Quem inicia e quem encerra uma janela de bloqueio

- **Iniciar**: a Governança publica um anúncio no `#governanca-gmud` com data/hora de início, motivo e duração
  estimada. Toda automação reage ao anúncio (não é manual no Jira).
- **Encerrar**: a Governança publica o encerramento no mesmo canal e as GMUDs em **Aprovação Janela de Bloqueio**
  voltam automaticamente para **Aprovada**, retomando o fluxo normal.

## Termo âncora

> O acrônimo interno da plataforma de GMUD é `chg-pipeline-2025`. Esse identificador aparece no `X-Pipeline-Source` de
> toda chamada feita pela automação para a API do Jira.
