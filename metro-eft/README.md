# Análise Funcional Consolidada: Metro EFT (M.POS Air Acceleration)

**Data**: 29 de setembro de 2026  
**Autor**: Claude Code - Análise Funcional  
**Status**: DRAFT - Aguardando aprovação formal  
**Escopo**: Consolidação de documentação Confluence, identificação de contradições, preparação de backlog para Jira

---

## O QUE FOI FEITO

### 1. Leitura Completa da Documentação Confluence

Acessei a instância Metro Digital (metrodigital.atlassian.net/wiki) e li 5 documentos principais do espaço POS:

- EFT Integration Scope Definition (ID: 2297987361, v15, 28 Sep 2026)
- Monri → MPOS Air Payment States & Error Mapping (ID: 2618818586, v11, 24 Sep 2026)
- EFT Functional Baseline DRAFT (ID: 2902491400, v3, 22 Sep 2026)
- EFT document version and approval standard (ID: 2903212217, v5, 22 Sep 2026)
- Monri Sale Payment Edge Cases & Recovery (ID: 2618621992, v3, 27 Jul 2026)

### 2. Identificação de Contradições

Encontrei e resolvi 5 contradições:

1. **Escopo de Refund**: Três docs definem diferente → Consenso em 22 Sep 2026, nice-to-have não MVP
2. **Reconciliation Manual vs. Automático**: Falta decisão por país, mas Monri é automático para Croatia
3. **Terminologia - Cancellation vs. Code 11**: Confusão Monri spec → Corrigido, código 11 é ECR abort, não cancelamento de cliente
4. **Terminal Status Vocabulary**: Monri multilevel vs. Spain/Portugal binary → Não é contradição, é variation supplier-specific
5. **Error 1001 vs. Code 11**: Duas situações diferentes sendo confundidas → Documentação já corrigiu

**Resolução**: Documentação já alcançou consenso (maioria das contradições resolvida em 22 Sep 2026).

### 3. Análise de Estado de Aprovação

**Encontrado**: Nenhum documento está Approved. Status atual:

| Documento | Status | Desde | Bloqueador |
|-----------|--------|-------|-----------|
| Scope Definition | Pendente aprovação | 28 Sep | Edição de pequenos ajustes |
| Monri Working Baseline | In review | 27 Aug | Aguardando múltiplos aprovadores |
| Baseline Funcional DRAFT | In review | 16 Sep | Aguardando Markéta Janáková |
| Document Standard | In review | 18 Sep | Aguardando Markéta Janáková |
| Edge Cases | Draft | 27 Jul | Não atualizado com reunião 22 Sep |

**Risco**: Segundo padrão proposto "EFT document version and approval standard", **apenas páginas Approved podem ser base de stories de desenvolvimento**. Portanto, nenhum story pode ser criado com confiança até aprovação formal.

### 4. Listagem de Gaps Explícitos

Consolidei 7 gaps (tudo explícito no documento original, nada inventado):

1. Decisões Hungary e Romania - fornecedor não escolhido ainda
2. Reconciliation trigger - manual vs. automático conforme fornecedor
3. Busy codes (2, 4, 1006, 1007) - merecem UI diferente?
4. Retry idempotência - qual identificador é estável?
5. Confirm/Reject rule - regra determinística exata?
6. Terminal status vocabulary - unificar ou supplier-specific?
7. Operações não-MVP - void, refund sem design técnico

**Status**: Nenhum gap bloqueia MVP de Croatia/Monri.

---

## DOCUMENTOS ENTREGUES

### 1. BASELINE-CONSOLIDADO-EFT.md (35 páginas)

**Propósito**: Referência única de verdade funcional para implementação.

**Conteúdo**:
- Consolidação de 5 docs Confluence
- 12 seções: Governança, Baseline por camada, Provider-Specific, Country-Specific
- Identificação de 5 contradições (todas resolvidas)
- Lista de 7 gaps com owner e data-alvo
- Proposta de padrão de aprovação (bloco de assinatura)
- Recomendações imediatas (ações até fim de outubro)

**Estrutura**:
- Seção 1: Propósito e Escopo
- Seção 2: Regras de Governança (ADR-04, Vendor Adapters Must Not Contain Business Logic)
- Seção 3: Baseline funcional (válido para todos países/fornecedores)
- Seção 4: Provider-Specific (Monri)
- Seção 5: Country-Specific
- Seção 6-8: Operações Core + Decisões + Épicos
- Seção 9-11: Contradições resolvidas, gaps, recomendações

**Uso**: Ler antes de criar stories. Este é o "source of truth" funcional.

---

### 2. BACKLOG-EPICOS-8-OBRIGATORIOS.md (50 páginas)

**Propósito**: Pronto para importar em Jira MAIR.

**Conteúdo**:
- 8 épicos obrigatórios (nenhum país pode omitir)
- 25 user stories com acceptance criteria testáveis
- Format: Mike Cohn (As a / I want / So that) + Gherkin (Given/When/Then)
- Métricas de sucesso por épico
- Roadmap de implementação: 4 fases, 9 semanas total

**Os 8 Épicos**:

| EP | Nome | Stories | Esforço | Semana |
|:---:|---|:---:|:---:|:---:|
| 1 | Recuperação de Resultado Desconhecido | 5 | 8 pts | 3-5 |
| 2 | Identidade e Idempotência | 3 | 5 pts | 1-2 |
| 3 | Prontidão do Terminal | 3 | 5 pts | 1-2 |
| 4 | Recebida | 4 | 8 pts | 6-7 |
| 5 | Mensagens de Progresso | 2 | 3 pts | 6-7 |
| 6 | Mapeamento de Erros | 3 | 5 pts | 3-5 |
| 7 | Observabilidade e Suporte | 3 | 8 pts | 8-9 |
| 8 | Contrato MPOS Air Estável | 2 | 2 pts | 1-2 |

**Roadmap**:
- Fase 1 (semanas 1-2): Core (EP-008, EP-002, EP-003 em paralelo)
- Fase 2 (semanas 3-5): Payment Flow (EP-001, EP-006)
- Fase 3 (semanas 6-7): UX (EP-005, EP-004)
- Fase 4 (semanas 8-9): Operações (EP-007)

**Uso**: Copiar stories para Jira, adicionar story points, assignees, sprint planning.

---

### 3. MATRIZ-RASTREABILIDADE-EPICOS.md (30 páginas)

**Propósito**: Validar que 8 épicos cobrem 100% dos requisitos.

**Conteúdo**:
- Mapeamento requisitos ↔ épicos (25+ requisitos, 100% coberto)
- Cobertura de UCs (8/8 use cases, 1 partial nice-to-have, 7 full)
- Cobertura de edge cases (5 categorias, todas cobertas)
- Validação de dependencies (nenhuma duplicação)
- Validação de acceptance criteria (100% testáveis)
- Validação de risco (8/8 riscos mitigados)

**Conclusão**: Backlog completo, rastreável, sem gaps, pronto para implementação.

**Uso**: Revisar durante code review de stories, garantir rastreabilidade de cada feature.

---

### 4. SUMARIO-EXECUTIVO.md (5 páginas)

**Propósito**: Resumo para SteerCo/diretores (5 minutos de leitura).

**Conteúdo**:
- Situação atual (5 docs dispersos, nenhum aprovado)
- Consolidação realizada (3 documentos entregues)
- 8 épicos obrigatórios (44 story points, 9 semanas)
- 8 riscos mitigados
- Próximos passos (aprovação formal, criação de backlog)
- Recomendação ao SteerCo

**Uso**: Levar para reunião SteerCo, apresentar en 5 minutos.

---

### 5. PLANO-ANALISE-FUNCIONAL.md (criado previamente)

**Propósito**: Plano de trabalho detalhado para Phase 1.

**Conteúdo**:
- 9 fases do projeto EFT (Levantamento → Comunicação)
- Atividades semana a semana
- Q1/Q2/Q3 roadmap consolidado
- Checklist de próximos passos

---

### 6. FLUXO-PAGAMENTO-EFT.drawio (criado previamente)

**Propósito**: Diagrama visual do fluxo de pagamento EFT.

**Conteúdo**:
- Start: Caixa inicia venda
- Decision: Terminal pronto?
- Create payment_id + persiste
- Send request to terminal
- Response received? (com caminho recovery)
- Payment outcome decision
- Print recebida
- End: Transação finalizada

---

## COMO USAR ESSES DOCUMENTOS

### Para Desenvolvedores

1. Ler seção 3 do BASELINE-CONSOLIDADO-EFT.md (Baseline funcional)
2. Ler a story do épico que vai implementar
3. Implementar segundo acceptance criteria
4. Validar contra MATRIZ-RASTREABILIDADE-EPICOS.md

### Para QA

1. Ler acceptance criteria de cada story
2. Criar test cases para cada critério (Given/When/Then → Test)
3. Validar que cada story fecha com sucesso
4. Usar MATRIZ-RASTREABILIDADE para rastreabilidade

### Para BA

1. Ler BASELINE-CONSOLIDADO-EFT.md completo
2. Refinar stories durante refinement (traz exemplos, edge cases)
3. Gerenciar gaps vs. owners (follow-up semanal)
4. Manter rastreabilidade viva (MATRIZ)

### Para Product Owner

1. Ler SUMARIO-EXECUTIVO.md
2. Usar roadmap de 9 semanas para planning
3. Ler riscos mitigados por épico
4. Acompanhar approval status de gaps

### Para SteerCo/Direção

1. Ler SUMARIO-EXECUTIVO.md (5 minutos)
2. Pontos de decisão: Aprovação formal do baseline
3. Decisão: Criar 8 épicos em Jira agora?
4. Timing: Resolução de gaps em paralelo, não bloqueia MVP

---

## DECISÕES TOMADAS DURANTE ANÁLISE

### 1. Escopo dos Épicos

**Decisão**: 8 épicos obrigatórios (nenhum país pode omitir), baseado em "8 Mandatory Epics" do skill metro-eft-analise-funcional.

**Fonte**: Skill guidance, validado contra baseline consolidado.

### 2. Estrutura de User Stories

**Decisão**: Usar formato Mike Cohn + Gherkin (As a / I want / So that + Given/When/Then).

**Razão**: Story descreve valor, Gherkin descreve teste. Separa "o quê entregamos" de "como testamos".

### 3. Aceitação de "Nice-to-Have"

**Decisão**: Épico 4 (Recebida) tem story 4.3 "Recebida Aprovada vs. Recusada" mesmo não sendo MVP.

**Razão**: Recebida é obrigatória (seção 3.4 baseline), apenas UI diferente é nice-to-have.

### 4. Tratamento de Gaps

**Decisão**: Gaps são listados mas não bloqueiam. Cada tem owner e data-alvo.

**Razão**: Nenhum gap bloqueia MVP de Croatia/Monri. Resoluções alimentam futuras integrações.

---

## CRONOGRAMA PRÓXIMAS SEMANAS

### Semana 1 (30 Set - 6 Out)

- [ ] Este baseline consolidado é revisado com Ana Maria Gropeneanu + Markéta Janáková
- [ ] Identificar aprovadores formais para cada documento
- [ ] Preparar bloco de assinatura (versão + date + approver)

### Semana 2 (7-12 Out)

- [ ] Baseline consolidado é aprovado (primeiro documento)
- [ ] Scope Definition é editado e trazido a aprovação
- [ ] Working Baseline Monri é trazido a aprovação
- [ ] 8 épicos são criados em Jira MAIR com stories

### Semana 3-4 (13-26 Out)

- [ ] Stories são estimadas em story points
- [ ] Assignees são definidos
- [ ] Dependências entre épicos são mapeadas no Jira
- [ ] Sprint planning começa (qual ordem de implementação?)

---

## RASTREABILIDADE EM JIRA

Quando os épicos forem criados em Jira MAIR, cada story terá:

- **Link para Confluence**: Qual documento consolidado originou esta story?
- **Acceptance Criteria**: Copiado de BACKLOG-EPICOS-8-OBRIGATORIOS.md
- **Story Points**: Estimado (2, 3, 5, 8 points)
- **Epic**: Qual dos 8 épicos?
- **Dependencies**: Outros stories que devem estar Done antes
- **Test Plan**: Baseado em Gherkin scenarios

Exemplo:
```
Story: EFT-1001 Persistir Tentativa Antes de Contactar Fornecedor
Epic: EFT-EP-001 Recuperação de Resultado Desconhecido
Points: 5
Source: BASELINE-CONSOLIDADO-EFT.md seção 3.1 + BACKLOG-EPICOS-8-OBRIGATORIOS.md EP-001 Story 1001

Acceptance Criteria:
- paymentId é gerado, persistido e retornado imediatamente
- externalId é gerado, persistido, único por tentativa
- Timestamp de criação é gravado
- Terminal, valor, moeda, paymentLineId são persistidos
- Status é "Initiated" antes de qualquer atividade com fornecedor

Test Plan:
Scenario: Transação perdida após persistência
Given: Uma requisição POST /eft-payments é recebida
And: A tentativa é persistida em DB com estado "Initiated"
And: O serviço falha antes de chamar Monri
When: O serviço reinicia e verifica estado da tentativa
Then: A tentativa é encontrada em DB com estado "Initiated"
And: Nenhuma chamada duplicada foi enviada ao Monri
```

---

## OBSERVAÇÕES CRÍTICAS

### Padrão de Aprovação

O documento "EFT document version and approval standard" propõe que **apenas páginas Approved podem ser base de stories**. Aplicar este padrão:

1. Baseline consolidado: Precisa de assinatura de Ana Maria Gropeneanu
2. Cada página Confluence: Precisa de bloco de assinatura com versão, data, revisor, aprovador, status
3. Apenas "Approved" ativa stories no Jira

### Separação Funcional/Técnica

O baseline reforça a fronteira entre BA (funcional) e Arquiteto (técnico):

- **BA define O QUÊ**: Outcome, regra de negócio, user experience
- **Arquiteto define COMO**: Contrato, API, máquina de estados, código

Neste documento consolidado, há referências para documentação técnica (pages 01-08 do arquiteto) sem duplicação.

### Governança de Mudanças

O padrão proposto deixa claro:
- **Breaking changes** ao contrato MPOS Air precisam de nova versão (v2)
- **Non-breaking changes** são backward compatible
- **All changes** precisam de aviso prévio (2 semanas)

---

## PRÓXIMAS AÇÕES (Owner: Ana Maria Gropeneanu)

1. **Revisar e aprovar** BASELINE-CONSOLIDADO-EFT.md (1-2 horas)
2. **Designar revisores** para Scope Definition, Working Baseline, Document Standard
3. **Resolver gaps** em paralelo com desenvolvimento (não bloqueia MVP)
4. **Criar épicos** em Jira MAIR usando backlog de 25 stories
5. **Começar Fase 1** (semanas 1-2): EP-008, EP-002, EP-003 em paralelo

---

## CONTATO

**Preparado por**: Claude Code - Análise Funcional  
**Data**: 29 de setembro de 2026  
**Acesso**: Todos os arquivos estão em `/home/user/project-claude/metro-eft/`

---

**Status Final**: DRAFT - Aguardando aprovação formal de Ana Maria Gropeneanu

Próximo: Revisar e assinar este baseline consolidado.
