# Plano de Análise Funcional - Metro EFT (M.POS Air)

**Data de Criação:** 29 de setembro de 2026  
**Status:** Planejamento  
**Responsável:** Business Analyst (CGI)  
**Stakeholders:** Metro Digital, CGI, Fornecedores de PSP

---

## Visão Geral do Projeto

O **M.POS Air Acceleration, Stream EFT e Fiscalização** é um projeto de modernização do sistema de pagamentos eletrônicos da Metro. O objetivo é:

1. Estabelecer um framework EFT reutilizável independente de fornecedor
2. Integrar múltiplos fornecedores de PSP (Payment Service Providers)
3. Garantir compliance fiscal e de segurança por país
4. Preparar onboarding eficiente para novos países

---

## Fases de Análise Funcional

### FASE 1: Levantamento de Baseline (Semanas 1-2)

**Objetivo:** Ler e consolidar documentação existente, identificar contradições, preparar perguntas críticas.

#### Atividades

1. **Leitura de Confluence (Confluence, Jira, Miro)**
   - Explorar espaço POS: documentação core, decisões registadas, roadmap
   - Ler espaços relacionados: MPOSAirPayment, MPA, MT
   - Extrair tabelas de gap por país e fornecedor
   - Verificar status de aprovação de documentos

2. **Inventário de Documentação**
   - [ ] Quantas páginas core existem? (Level 1, 2)
   - [ ] Quantas páginas vendor-specific? (Level 3)
   - [ ] Quantos países estão documentados?
   - [ ] Quantos fornecedores integrados?
   - [ ] Qual é o estado de cada documento? (Draft, Review, Approved, Outdated)

3. **Caça a Contradições**
   - Comparar escopo declarado vs. decisões registadas
   - Verificar identidade e idempotência em pagamentos
   - Identificar vocabulário inconsistente de estados do terminal
   - Detectar capacidades marcadas como "planeadas" vs. "piloto"
   - Listar endpoints sem descrição funcional

4. **Perguntas Críticas a Registar**
   - [ ] Qual é o identificador único de uma tentativa de pagamento?
   - [ ] Qual é a cadeia de recuperação se a resposta for perdida após despacho?
   - [ ] Como é feita a deduplica­ção de transações?
   - [ ] Qual é a regra de impressão de recibo por país?
   - [ ] Quem decide se uma transação é retentável?

#### Artefatos de Saída (Fase 1)

- **Checklist de Documentação**: Estado de cada página
- **Matriz de Contradições**: O que diverge entre documentos
- **Banco de Perguntas**: 20-30 perguntas para Metro e fornecedores
- **Roadmap de Clarificações**: Prioridade de respostas

---

### FASE 2: Baseline Funcional Reutilizável (Semanas 3-4)

**Objetivo:** Escrever a especificação funcional core que vale para todos os países.

#### Elementos Obrigatórios de Cada Documento Core

1. **Identificação**
   - Nome do documento
   - Nível: Level 1 (Foundations) ou Level 2 (Core Framework)
   - País: [GLOBAL]
   - Fornecedor: [AGNOSTIC]
   - Versão: X.Y
   - Aprovador: Nome, Data
   - Estado: Draft / Review / Approved / Outdated

2. **Escopo Baseline**
   - Fluxos de pagamento (autorização, confirmação, negação, reversão)
   - Comportamento do terminal (estados permitidos, transições)
   - Recuperação de resultado desconhecido (a rotina mais crítica)
   - Idempotência e deduplica­ção
   - Mapeamento de erros (apenas as chaves normalizadas)
   - Recibo normalizado

3. **Fora de Escopo (Explícito)**
   - Reconciliação (específica por fornecedor)
   - DCC (Conversão de Moeda Dinâmica)
   - Programas de fidelização
   - Multi-adquirente

4. **Referências Cruzadas**
   - Link para documentação técnica (nunca copiar)
   - Link para páginas vendor-specific
   - Link para decisões registadas em Jira

#### User Stories de Baseline (Fase 2)

```
Story 1: Estabelecer Vocabulário Único de Estados do Terminal
As a developer implementing a new vendor adapter,
I want to know the complete vocabulary of terminal states (READY, BUSY, ERROR_COMM, etc),
so that I can map vendor-specific states to normalized states without ambiguity.

Acceptance Criteria:
- Scenario: New vendor integration begins
- Given I have the baseline documentation
- When I check the "Terminal States" section
- Then I see a table with: state name, description, what actions are allowed, transitions, and example codes from at least one existing vendor
- And the table is marked "Level 2 Core Framework, applies to all vendors and countries"
```

```
Story 2: Document the Recovery Flow for Unknown Outcomes
As a support engineer,
I want to know exactly how to recover from a situation where a payment was sent to the terminal but the response was lost,
so that I can tell a store manager exactly what happened to their payment.

Acceptance Criteria:
- Scenario: Payment sent, response lost, customer asks "did I get charged?"
- Given I have the recovery documentation
- When I follow the recovery procedure
- Then I can confirm the payment state using the payment ID and terminal ID
- And the payment either shows as "completed" or "not attempted"
- And recovery never retries the payment, only checks state
```

#### Artefatos de Saída (Fase 2)

- **Documento Core 1: Payment Flow & Terminal States** (Level 2)
- **Documento Core 2: Error Mapping Framework** (Level 2)
- **Documento Core 3: Unknown Outcome Recovery** (Level 2)
- **Documento Core 4: Idempotency & Deduplication** (Level 2)
- **Documento Core 5: Receipt Format Normalization** (Level 2)
- **Checklist de Aprovação**: Assinado por Metro e CGI

---

### FASE 3: Matriz de Gap por País/Fornecedor (Semanas 5-6)

**Objetivo:** Para cada país novo, definir apenas o que muda em relação ao baseline.

#### Processo de País Novo (Playbook)

**Fase 0: Requisitos**
- Enviar checklist de requisitos ao fornecedor (10 áreas)
- Criar página de stakeholders (vazia, para preencher)
- Criar página-mãe do fornecedor (vazia, para preencher)

**Fase 1: Preencher Tabela de Gap**
- Tabela com colunas: Capacidade | Baseline | Gap | Decisão
- Exemplo de gap: "Void de transações" (baseline: permite) → "Não permitido neste país" → Decisão: "App recusa antes de enviar"

**Fase 2: Decisões de País**
- Void? (sim/não)
- Refund? (sim/não)
- Pré-autorização? (sim/não)
- Cashback? (sim/não)
- Fidelização? (sim/não)
- DCC? (sim/não)
- Prestações? (sim/não)
- Multi-adquirente? (sim/não)

**Fase 3: Mapeamento de Erros**
- Tabela: Código Fornecedor | Categoria Arquitetura | Chave Normalizada | Mensagem Unificada
- Exemplo: "05 Declined" → "Declined" → "PAYMENT_DECLINED" → "Seu pagamento foi recusado. Verifique seus dados."

**Fase 4: Recibo**
- Regra legal do país (quantas cópias, informações obrigatórias)
- Comportamento em falha de impressão
- Comportamento em timeout de impressão

**Fase 5: Reconciliação**
- É automática? (sim) → sistema verifica sem ação humana
- É manual? (sim) → quem dispara, em que horário

#### User Stories de País Novo

```
Story 3: Onboard País Novo (Spain)
As a business analyst,
I want to onboard Spain with the minimum viable documentation,
so that implementation can begin without waiting for exhaustive specs.

Acceptance Criteria:
- Scenario: Spain is the next country
- Given I have the baseline documentation
- When I create the Spain page
- Then it contains only a gap table against baseline
- And each gap row has: baseline capability, Spain decision, and expected behavior
- And the gap table has fewer than 20 rows (if more, baseline is incomplete)
- And no content is duplicated from baseline
- And approval is signed by Metro ES stakeholder
```

#### Artefatos de Saída (Fase 3)

- **Documento Template: Country Page** (para reutilizar por país)
- **Documento Template: Gap Table** (com exemplos preenchidos)
- **Documento Template: Error Mapping** (colunas fixas)
- **Checklist de Onboarding**: 6 fases, critérios de entrada/saída

---

### FASE 4: Decisões Pendentes & Backlog (Semanas 7-8)

**Objetivo:** Registar decisões abertas, mapear epicos obrigatórios, estruturar backlog do Jira.

#### Épicos Obrigatórios (8 Épicos)

Estes épicos precisam de estar no backlog de qualquer integração EFT:

**Épico 1: Recuperação de Resultado Desconhecido**
- Quando: Resposta perdida após despacho, confirmação perdida, reinício com pagamento em curso
- Como: Por identificador de pagamento, nunca repetição cega
- Story: "Investigar pagamento por ID retorna estado definitivo"

**Épico 2: Identidade e Idempotência**
- Quando: Identificador único antes de cada tentativa, persistido antes de chamar fornecedor
- Como: Retry de transporte devolve estado existente, nunca segunda venda
- Story: "Request com payment_id repetido devolve resposta anterior"

**Épico 3: Prontidão do Terminal**
- Quando: Só iniciar venda se terminal em estado READY
- Como: Vocabulário único de estados, bloqueio no app
- Story: "Venda recusada com mensagem 'Terminal ocupado' se BUSY"

**Épico 4: Recibo**
- Quando: Dados normalizados, mascaramento de PAN, regra local
- Como: Impressora falha → comportamento definido por país
- Story: "Recibo imprime com máximo 4 dígitos do cartão visíveis"

**Épico 5: Mensagens de Progresso**
- Quando: Terminal emite eventos, app é dono do texto
- Como: Chaves normalizadas, localização centralizada
- Story: "Caixa vê 'Processando pagamento...' em linguagem do país"

**Épico 6: Observabilidade & Suporte**
- Quando: Procura por qualquer ID, sequência de eventos, runbooks
- Como: Logs com carimbos, sem dados sensíveis
- Story: "Suporte encontra pagamento por transaction_id em < 10s"

**Épico 7: Mapeamento de Erros**
- Quando: Código fornecedor → chave normalizada → mensagem unificada
- Como: Tabela reutilizável, extensível por país
- Story: "Novo código de erro mapeado em < 1 dia sem redeployment"

**Épico 8: Integração com Novo Fornecedor**
- Quando: PSP novo entra no portfolio
- Como: Adapter vendor-specific, não toca baseline
- Story: "Novo fornecedor integrado com <= 20% código novo"

#### Matriz de Rastreabilidade

Ligar cada: Documento Core → Decisão → Épico → Story → Caso de Teste

Exemplo:
```
Documento: "Unknown Outcome Recovery" (Level 2)
  ├─ Decisão: "Recovery nunca retenta, só confirma" (Jira MAIR-123, aberta há 30 dias)
  ├─ Épico: "Recuperação de Resultado Desconhecido" (MAIR-450)
  │  ├─ Story: "Investigar pagamento por ID" (MAIR-451)
  │  └─ Caso de Teste: "TC-ERR-001: Payment lost, recovery succeeds"
  └─ Referência Cruzada: Técnico "API GET /payments/{id}" (Confluence MT-45)
```

#### User Stories de Decisões

```
Story 4: Registar & Resolver Decisão "Retentabilidade"
As a product owner,
I want to have a single, auditable place where I register who decides whether a payment is retryable,
so that support, development, and vendors all follow the same rule.

Acceptance Criteria:
- Scenario: New vendor asks "if DECLINED, can we retry?"
- Given I have the decision log
- When I check the "Retentability Rule" decision
- Then I see: decision text, who decided (Metro role), date decided, and which epic implements this
- And the decision is older than 30 days (so it's been tested in production)
- And support can cite this decision to the vendor
```

#### Artefatos de Saída (Fase 4)

- **RAID Log** (Risks, Assumptions, Issues, Dependencies) no Jira
- **Banco de Decisões** (30-50 decisões com idade, bloqueadores)
- **Matriz de Rastreabilidade** (doc → decisão → épico → story → teste)
- **Backlog Estruturado do Jira**:
  - 8 Épicos obrigatórios (MAIR-450 a MAIR-457)
  - 40-60 Stories (divididas por épico)
  - Definition of Ready & Definition of Done preenchidas

---

### FASE 5: Plano de Comunicação & Governança (Semana 9)

**Objetivo:** Estruturar como o projeto reporta ao nível executivo (SteerCo).

#### Artefatos Vivos (Atualizar a cada sprint)

1. **Painel de Controlo**
   - Documentos pendentes (quais, quem, prazo)
   - Decisões com idade > 30 dias (bloqueadores)
   - Stories bloqueadas (por quê)
   - Saúde geral do baseline (% completo, % aprovado)

2. **Banco de Perguntas**
   - Pergunta | Data | Destinatário | Resposta | Página onde ficou registada
   - Meta: Todas as perguntas respondidas em < 7 dias

3. **Matriz de Saúde do Baseline**
   - Por área (Terminal States, Error Mapping, Recovery, Receipt, Reconciliation)
   - Estado: Red (não começou) | Yellow (em progresso) | Green (aprovado)
   - Risco: O que pode adiar

4. **Radar de Países**
   - País | Fornecedor | Fase (0-6) | Bloqueador | ETA onboarding

#### Material para SteerCo (Mensal)

```
SLIDE 1: Progresso de Baseline
- Baseline X% aprovado (verde se > 80%)
- Decisões pendentes (número, idade média)
- Perguntas respondidas (X de Y em tempo)

SLIDE 2: Saúde Operacional
- Quantos países prontos para onboarding?
- Quantos fornecedores integrados?
- Qual é o custo incremental do país seguinte?

SLIDE 3: Radar de Riscos
- Riscos técnicos: [listar]
- Riscos de negócio: [listar]
- Dependências externas: [listar]
- Mitiga­ções: [listar]

SLIDE 4: Próximas Ações
- Decisão solicitada: [X]
- Escalações necessárias: [Y]
- Recursos adicionais: [Z]
```

#### Artefatos de Saída (Fase 5)

- **Painel de Controlo** (Jira Dashboard)
- **Banco de Perguntas** (Confluence + Rastreabilidade)
- **Roadmap de Países** (Prioridade, Sequência, ETA)
- **Template de SteerCo** (Deck mensal, métricas fixas)

---

## Roadmap de Trabalho Consolidado

### Trimestre 1 (Q1 2026): Baseline & Decisões

| Semana | Fase | Entregáveis | Responsável | Bloqueadores |
|--------|------|------------|-------------|-------------:|
| 1-2 | Levantamento | Checklist doc, Banco de perguntas | BA | Acesso Confluence |
| 3-4 | Baseline Core | 5 docs Core, User Stories | BA + Tech Lead | Respostas às perguntas |
| 5-6 | Gap & Onboarding | Template país, Gap table | BA | Feedback Metro |
| 7-8 | Decisões | RAID log, 8 Épicos | BA + PO | Reuniões SteerCo |
| 9 | Comunicação | Painel, SteerCo material | BA | Alinhamento exec |

### Trimestre 2 (Q2 2026): Implementação & Primeiro País

| Semana | Fase | Entregáveis | Responsável | Bloqueadores |
|--------|------|------------|-------------|-------------:|
| 1-4 | Dev Épico 1-2 | Recovery flow + Idempotência | Dev + QA | Aprovação Baseline |
| 5-8 | Dev Épico 3-4 | Terminal States + Receipt | Dev + QA | Aprovação Baseline |
| 9-12 | Onboarding País 1 | Gap table, Vendor mapping, Testes | BA + Dev | Respostas vendor |
| 13 | Demo & Feedback | Demonstração, Ajustes | PM | Aceitação Metro |

### Trimestre 3+ (Q3+): Scaling & Países Subsequentes

| Fase | Objetivo | Métrica Sucesso |
|------|---------|-----------------|
| Scaling | Reduzir custo de país novo de 40 dias para 20 dias | País novo em <= 20 dias |
| Portfolio | 10+ países, 5+ fornecedores | Zero contradições, 100% gap table preenchida |
| Suporte | Runbooks completos, suporte em 4 idiomas | Tempo resolução ticket < 8h |

---

## Métricas de Sucesso

### Fase 1 (Levantamento)
- ✓ 100% documentação catalogada
- ✓ 50+ contradições identificadas e ranqueadas
- ✓ 30 perguntas criadas, respondidas em <= 7 dias

### Fase 2 (Baseline)
- ✓ 5 documentos core aprovados por Metro e CGI
- ✓ 0 suposições em documentação (tudo verificado)
- ✓ 100% rastreabilidade doc → decisão

### Fase 3 (Gap)
- ✓ Template país reduz onboarding de 40 dias para 20 dias
- ✓ Primeiro país testa template, 0 retrabalho
- ✓ Métrica: país novo <= 20% conteúdo novo (baseline sucesso)

### Fase 4 (Backlog)
- ✓ 8 épicos obrigatórios no backlog com story points
- ✓ 60 stories com critérios de aceitação testáveis
- ✓ 0 stories bloqueadas quando sprint inicia

### Fase 5 (Comunicação)
- ✓ Painel atualizado 2x por semana
- ✓ SteerCo tem visibilidade de riscos e decisões com < 48h de latência
- ✓ 100% das perguntas respondidas em <= 7 dias

---

## Riscos & Mitigações

| Risco | Impacto | Probabilidade | Mitiga­ção |
|-------|---------|---------------|-----------:|
| Documentação incompleta no Confluence | Alto: Atrasa baseline | Média | Revisor dedicado, checklist semanal |
| Decisões lentas de Metro | Alto: Bloqueia desenvolvimento | Média | Escalação SteerCo semanal, SLA 7 dias |
| Vendor não responde a requisitos | Médio: Atrasa país | Alta | Enviar checklist na Fase 0, follow-up 48h |
| Contradições descubiertas em prod | Alto: Retrabalho crítico | Baixa | Testes E2E rigorosos, campo de gap antes de prod |
| Mudança de escopo mid-projeto | Médio: Atrasa baseline | Média | Governance clara, decisões registadas, revisões mensais |

---

## Próximos Passos Imediatos

### Semana 1

- [ ] Agendar acesso ao Confluence (metrodigital.atlassian.net/wiki)
- [ ] Agendar reunião com Product Owner Metro (escopo, aprovadores)
- [ ] Agendar reunião com Tech Lead CGI (constraints técnicas)
- [ ] Criar página de "Levantamento em Curso" no Confluence
- [ ] Começar leitura do espaço POS

### Semana 2

- [ ] Completar checklist de documentação (estado de cada página)
- [ ] Listar 10 contradições mais críticas
- [ ] Preparar banco de perguntas (20-30 perguntas)
- [ ] Agendar sessão de caça a contradições com Tech Lead
- [ ] Propor prioridade de respostas a Metro

### Semana 3

- [ ] Começar primeiro documento core (Terminal States)
- [ ] Enviar perguntas a Metro e principais vendors
- [ ] Criar stories de baseline (primeiro draft)
- [ ] Revisar com CGI tech team

---

## Contatos & Escalações

| Papel | Nome | Escalação |
|-------|------|-----------|
| Product Owner (Metro) | [TBD] | [TBD Manager] |
| Tech Lead (CGI) | [TBD] | [TBD Manager] |
| Vendor Manager | [TBD] | PO Metro |
| Aprovador Baseline | [TBD] | SteerCo |

---

**Documento Vivo:** Atualizar com feedback de stakeholders a cada 2 semanas.

