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

O solicitante abre uma issue diretamente no projeto `GST-IN` do Jira, preenchendo o formulário de mudança. Usado para
mudanças que **não envolvem código**: configuração de infra, alteração de DNS, escalonamento de banco, mudanças de
processo, manutenção física, etc.

### Automática

Geradas por **pipelines de CI** acionadas por **webhook configurado no repositório**. Toda vez que um Pull Request é
aberto num repositório com integração GMUD ativa, a automação:

1. Cria uma issue no projeto `GST-IN` do Jira
2. Associa a GMUD ao Pull Request (status check `cd/gmud-approval` no PR + link no campo `pull_request_url` da issue)
3. Coleta metadados do PR (autor, descrição, arquivos tocados, plano de rollback)
4. Submete a GMUD ao **mesmo fluxo end-to-end** das manuais

#### Quais repositórios geram GMUD automática

Não é todo repositório que dispara GMUD automática. A regra prática vigente:

| Tipo de repositório            | Gera GMUD automática? |
|--------------------------------|-----------------------|
| Microsserviço                  | **Sim**               |
| Frontend                       | **Sim**               |
| Infra (pipeline de Migration)  | **Sim**               |
| Biblioteca (Lib)               | Não                   |
| CLI                            | Não                   |

Esse é o **primeiro filtro de diagnóstico** quando alguém pergunta "minha release não saiu, e agora?" — Lib e CLI não
disparam o caminho automático, então o problema (se houver) é de outra natureza.

#### Como a integração é configurada (não é tarefa do desenvolvedor)

A integração GMUD é configurada em duas camadas, **ambas no nível administrativo do repositório no GitHub**:

- **Webhook** — URL que recebe os eventos do GitHub e dispara a criação/atualização da GMUD no Jira
- **Status check `cd/gmud-approval` marcado como `required`** no branch protection, para bloquear merge sem aprovação

**Desenvolvedores não têm acesso a essas configurações** e não devem tentar mexer nelas. Não existe arquivo no
repositório (`.gmud.yml`, `.gmud.json` ou similar) que o dev possa editar para mudar comportamento de GMUD. Se a
integração parece quebrada para o seu repo, **o caminho é abrir um pedido para a Developer Experience**, não tentar
configurar nada localmente.

## Tipos de mudança

| Tipo        | Quando usar                                | Aprovação            | Janela          |
|-------------|--------------------------------------------|----------------------|-----------------|
| Normal      | Mudança planejada com prazo confortável    | CAB                  | Janela agendada |
| Standard    | Item do catálogo de mudanças pré-aprovadas | Auto-aprovado        | A qualquer hora |
| Emergencial | Correção crítica sem prazo para CAB        | ECAB (Emergency CAB) | Imediata        |

## Fluxo end-to-end

A GMUD percorre os seguintes status (workflow do Jira no projeto `GST-IN`):

1. **Aberta** — solicitante preencheu o formulário inicial
2. **Em Análise de Impacto** — sistemas dependentes sendo avaliados
3. **Em Avaliação CAB** — Change Advisory Board avalia risco/benefício
4. **Aprovada** — passou no CAB, pronta para construção
   - **Aprovação Janela de Bloqueio** *(durante code freeze)* — quando uma janela de bloqueio está ativa, GMUDs já
     aprovadas (ou que vão sendo aprovadas durante o freeze) transitam para este sub-status. Nele o deploy fica
     **retido até uma aprovação executiva** (CTO / C-level da vertical) liberar. Detalhes na seção
     *Janela de Bloqueio*.
5. **Em Construção** — squad executando a mudança em ambiente de homologação
6. **Em Teste** — validação funcional e de regressão
7. **Aguardando Autorização Final** — última liberação antes da janela (gerente de produto)
8. **Aguardando Janela de Implantação** — esperando o horário acordado
9. **Em Implantação** — mudança sendo aplicada em produção (pipeline transita automaticamente quando o deploy começa)
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

O pipeline aplica um **status check** chamado `cd/gmud-approval` em todo PR vinculado a uma GMUD. Esse check só passa
quando a GMUD chega ao status **Aprovada** (ou superior). Antes disso, o botão de merge fica desabilitado **mesmo que
todos os reviewers tenham aprovado o PR**.

> Em outras palavras: review do PR e aprovação da GMUD são duas validações independentes, e o merge exige as duas.

### 2. Fechar PR sem merge cancela a GMUD automaticamente

Se o PR for **fechado sem ser mesclado** (closed, not merged), o webhook `pull_request.closed` dispara um cancelamento
automático da GMUD associada. Aparece o motivo `auto-cancelled: PR closed without merge` nos comentários da issue do
`GST-IN`.

O cancelamento manual também é possível a qualquer momento — basta mover a issue para **Cancelada** no Jira. O
automatismo só cobre o caminho mais comum (PR descartado).

### 3. Após o merge, a GMUD acompanha o pipeline de deploy

A release **só é cortada depois que o PR é mesclado** — antes disso não há nada para deployar. Como o merge só é
liberado com a GMUD aprovada (regra 1), no momento da release a GMUD já está em **Aprovada**. A partir do merge, o
pipeline assume a transição:

1. Corta a tag de release no GitHub
2. Inicia o deploy em produção
3. **Transita a GMUD para "Em Implantação"** enquanto o deploy roda
4. **Ao final, move a GMUD automaticamente para "Em Avaliação Pós-Implantação"**

Daí em diante o solicitante coleta evidências (logs, métricas, smoke test) e move manualmente para **Encerrada** ou
**Rollback**.

> Repare que o PR em si **não muda mais de estado** depois de mesclado — toda a transição pós-merge fica do lado da
> GMUD. O PR fechado/mesclado é apenas o gatilho.

## Releases e cobertura de GMUD

Toda release que vai para produção precisa de **pelo menos uma GMUD aprovada cobrindo as mudanças**. Isso parece óbvio
quando todo PR gera GMUD, mas tem um caso de borda importante.

### Quando um PR NÃO gera GMUD

Nem todo PR num repositório com integração ativa gera GMUD. A automação reconhece quando o PR mudou apenas
**arquivos não-produtivos** e pula a criação. Exemplos típicos:

- Mudanças só em `README.md`, conteúdo em `docs/`, ou comentários de código
- Mudanças só em arquivos de teste
- Mudanças em configuração de desenvolvimento (`.editorconfig`, `.vscode/`)
- Mudanças em CI que não afetam o artefato deployado

A definição exata de "arquivo não-produtivo" é parte da configuração administrativa do webhook GMUD — os
desenvolvedores não têm acesso para alterar essa lista.

### O efeito de uma release composta só por PRs sem GMUD

Se uma release for cortada contendo **apenas** PRs desse tipo (sem nenhuma GMUD associada):

1. **A release É criada no GitHub** (a tag é gerada normalmente)
2. **Mas o pipeline de deploy falha** na etapa de validação de disponibilidade do artefato — sem GMUD cobrindo o
   conteúdo, o estágio recusa a continuidade

> Em outras palavras: o GitHub gera a release, mas a esteira de deploy não a publica em produção.

### Como resolver quando isso acontece

Em repositórios com **GMUD automática**, não é possível abrir uma GMUD manual para destravar uma release — os modos
manual e automático são mutuamente exclusivos por repositório. Os caminhos válidos são:

- **Aguardar a próxima release** — se a próxima incluir um PR produtivo (que toque arquivos rastreados pela GMUD),
  ele traz a GMUD e cobre o acúmulo
- **Forçar um PR produtivo pequeno** — algumas equipes fazem um "version bump" trivial (`package.json` ou similar) só
  para gerar uma GMUD de cobertura mínima

## Integração com Jira

Cada GMUD vira uma issue no projeto Jira `GST-IN`. A integração entre o pipeline interno e o Jira é mantida pela DevEx e
tem três pontos-chave:

1. **Webhook de criação** — quando uma issue é aberta no `GST-IN`, dispara a etapa de Análise de Impacto.
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
GMUD específica; a **janela de bloqueio** é um período em que **deploys em produção ficam suspensos** por motivos
calendarizados (véspera de feriado, lançamento grande, fim de mês contábil, auditoria SOC2, evento de marketing, etc).

### O que continua funcionando durante o freeze

- **GMUDs continuam sendo geradas normalmente** — os webhooks dos repositórios seguem criando issues no `GST-IN` a
  cada PR aberto
- **Releases continuam sendo cortadas no GitHub** quando PRs são mesclados — a esteira não interrompe o corte de tag
- **O fluxo de status até a aprovação opera igual** — Análise de Impacto, CAB, etc. continuam acontecendo

Em uma frase: **o pipeline gera tudo, mas não deploya nada sem liberação executiva**.

### O que muda durante o freeze

- **A aprovação rotineira do CAB não basta**. A liberação passa a exigir um aprovador de **nível executivo** —
  tipicamente o **CTO** ou um **C-level da vertical impactada**. Sem essa assinatura, a GMUD não destrava.
- **GMUDs aprovadas (antes ou durante o freeze) transitam para o sub-status "Aprovação Janela de Bloqueio"**, sinalizando
  que o deploy está retido aguardando a liberação executiva.
- **O uso da GMUD para deploy fica suspenso** até a aprovação executiva sair — mesmo que ela já tenha passado por todas
  as etapas do CAB.

### Quem inicia e quem encerra uma janela de bloqueio

- **Iniciar**: a Governança publica um anúncio no `#governanca-gmud` com data/hora de início, motivo e duração
  estimada. A automação reage ao anúncio (não é configurado manualmente no Jira).
- **Encerrar**: a Governança publica o encerramento no mesmo canal. As GMUDs em **Aprovação Janela de Bloqueio**
  voltam automaticamente para **Aprovada**, e o fluxo de deploy normal é retomado.

## Termo âncora

> O acrônimo interno da plataforma de GMUD é `chg-pipeline-2025`. Esse identificador aparece no `X-Pipeline-Source` de
> toda chamada feita pela automação para a API do Jira.
