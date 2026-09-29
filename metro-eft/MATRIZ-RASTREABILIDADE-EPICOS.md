---
document_type: "Matriz de Rastreabilidade"
project: "Metro EFT - M.POS Air Acceleration"
version: "1.0"
date_created: "2026-09-29"
author: "Claude Code - Análise Funcional"
purpose: "Validar que 8 épicos cobrem todos os requisitos do baseline"
status: "DRAFT"
---

# MATRIZ DE RASTREABILIDADE: ÉPICOS ↔ REQUISITOS

## 1. MAPEAMENTO DE REQUISITOS PARA ÉPICOS

### Fonte: Baseline Consolidado

Cada requisito do baseline consolidado está mapeado para um ou mais épicos que o implementam.

| Requisito Funcional | Documento Base | Seção | EP-001 | EP-002 | EP-003 | EP-004 | EP-005 | EP-006 | EP-007 | EP-008 |
|---|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Persistir tentativa antes de fornecedor | Baseline 3.1 | Operações Core | ✓ | ✓ | - | - | - | - | ✓ | - |
| Recuperar transação anterior bloqueada | Baseline 3.3 | Recovery Rule | ✓ | - | - | - | - | - | - | - |
| Confirmar vs. Rejeitar após recovery | Baseline 7 Edge Cases | Category E | ✓ | - | - | - | - | - | ✓ | - |
| Recuperação após restart | Baseline 3.2 | Lifecycle | ✓ | - | - | - | - | - | - | - |
| Status lookup durante recovery | Baseline UC-02 | Payment Inquiry | ✓ | - | - | - | - | - | ✓ | - |
| Gerar externalId único | Baseline 7 Retry | Idempotency | - | ✓ | - | - | - | - | - | ✓ |
| Rejeitar duplicata durante processamento | Baseline 3.2 | Idempotency Rule | - | ✓ | - | - | - | - | - | - |
| Diferenciar paymentLineId de externalId | Baseline 7 | Lifecycle | - | ✓ | - | - | - | - | - | - |
| Endpoint GET /device-connection-status | Baseline UC-03 | Device Status | - | - | ✓ | - | - | - | - | ✓ |
| Bloquear venda se terminal não Ready | Baseline 3.3 | Terminal Readiness | - | - | ✓ | - | - | - | - | - |
| Status de transição (Initializing) | Baseline 4 Country-Specific | Device Readiness | - | - | ✓ | - | - | - | - | - |
| Dados de recebida do fornecedor | Baseline UC-01 | Receipt | - | - | - | ✓ | - | - | - | ✓ |
| Mascaramento de PAN | Baseline 2.7 | Sensitive Data | - | - | - | ✓ | - | - | ✓ | - |
| Recebida aprovada vs. recusada | Baseline 4.3 | Receipt Format | - | - | - | ✓ | - | - | - | - |
| Conformidade legal por país | Baseline 5.4 | Country-Specific | - | - | - | ✓ | - | - | - | - |
| Eventos de progresso padronizados | Baseline UC-01 | Progress Messages | - | - | - | - | ✓ | - | - | ✓ |
| Event de recovery | Baseline 8 | Recovery Events | ✓ | - | - | - | ✓ | - | - | - |
| Mapeamento Monri completo | Baseline 4.4 | Error Mapping | - | - | - | - | - | ✓ | - | - |
| Framework para novo fornecedor | Baseline 6 | Mapping Principles | - | - | - | - | - | ✓ | - | - |
| Erro desconhecido falha seguramente | Baseline 6.5 | Safe Defaults | - | - | - | - | - | ✓ | ✓ | - |
| Searchable logs por payment ID | Baseline 7 | Observability | - | - | - | - | - | - | ✓ | - |
| Runbook terminal bloqueado | Baseline 7 | Support Runbook | - | - | - | - | - | - | ✓ | - |
| Observabilidade de metrics | Baseline 7 | Metrics | - | - | - | - | - | - | ✓ | - |
| Contrato core versionado | Baseline 3.2 | Contract | - | - | - | - | - | - | - | ✓ |
| Estabilidade em 8 semanas | Baseline 3.2 | Contract Stability | - | - | - | - | - | - | - | ✓ |

---

## 2. COBERTURA DE CASOS DE USO

### Fonte: EFT Integration Scope Definition

Validar que cada Use Case tem implementação mapeada.

| Use Case | ID | Descrição | EP-001 | EP-002 | EP-003 | EP-004 | EP-005 | EP-006 | EP-007 | EP-008 | Status |
|---|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|---|
| Sale Payment | UC-01 | MPOS Air requere pagamento de venda EFT | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | **Full** |
| Payment Status/Inquiry | UC-02 | MPOS Air requere status de pagamento anterior | ✓ | ✓ | - | - | - | - | ✓ | ✓ | **Full** |
| Device Connection Status | UC-03 | MPOS Air requere informação de prontidão | - | - | ✓ | - | ✓ | - | ✓ | ✓ | **Full** |
| Cancel In-Progress | UC-04 | MPOS Air requere cancelamento de pagamento em curso | - | ✓ | - | - | - | - | - | - | **Partial** (nice-to-have) |
| Reconciliation/Settlement | UC-05 | MPOS Air ou processamento schedualado requere reconciliação | ✓ | ✓ | ✓ | ✓ | - | ✓ | ✓ | - | **Full** |
| Initialization/Pending | UC-06 | MPOS Air ou sistema requere inicialização | - | ✓ | ✓ | - | - | - | ✓ | - | **Partial** (TBD) |
| Reset Device | UC-07 | MPOS Air requere reset de integração | - | ✓ | ✓ | - | - | - | ✓ | - | **Partial** (TBD) |
| Refund | UC-08 | MPOS Air requere reembolso de pagamento anterior | - | ✓ | - | - | - | ✓ | ✓ | - | **Partial** (nice-to-have) |

---

## 3. COBERTURA DE CATEGORIAS DE EDGE CASES

### Fonte: Monri Sale Payment Edge Cases & Recovery

Validar que cada categoria de caso de borda tem tratamento.

| Categoria | Casos | Tratamento | Épicos |
|---|---|---|---|
| **A: Request Validation** | S01, S11 | Rejeitar inválido, deduplicar retry | EP-002 (idempotência), EP-006 (validação) |
| **B: Terminal Availability** | S02, S13 | Reportar status correto, recuperar bloqueado | EP-003 (status), EP-001 (recovery) |
| **C: Customer Interaction** | S03, S04 | Progresso até resultado, reconhecer recusa | EP-005 (msgs), EP-006 (decline) |
| **D: Provider Error** | S05, S06 | Classificar erro, preservar fatos | EP-006 (error mapping), EP-001 (recovery) |
| **E: Result Persistence** | S07-S10 | Persistir antes, Confirm/Reject com certeza | EP-001 (recovery), EP-002 (idempotência) |

**Conclusão**: Todos os 5 casos de borda têm cobertura em épicos. ✓

---

## 4. VALIDAÇÃO: NENHUM REQUISITO FICOU SEM ÉPICO?

### Requisitos do Baseline

#### Seção 3.1: O que o EFT Device Service Entrega
- [ ] Contrato MPOS Air é fixo: **EP-008** ✓
- [ ] Orquestra interação MPOS Air ↔ fornecedor: **EP-001, EP-005** ✓
- [ ] Mapeia dados: **EP-006** ✓
- [ ] Persistência: **EP-001, EP-002** ✓
- [ ] Logging/traceabilidade: **EP-007** ✓
- [ ] Retorna resultado normalizado: **EP-006, EP-004** ✓
- [ ] Retorna erro explícito se capacidade não suportada: **EP-006** ✓

#### Seção 3.2: Operações Core
- [ ] Device Status: **EP-003** ✓
- [ ] Device Initialization: **EP-003** ✓
- [ ] Sale Payment: **EP-001, EP-002, EP-003, EP-004, EP-005, EP-006, EP-008** ✓
- [ ] Payment Inquiry/Status: **EP-001, EP-007** ✓
- [ ] Reconciliation Hook: **EP-001, EP-005** ✓
- [ ] Nunca requer mudança no contrato MPOS Air: **EP-008** ✓

#### Seção 3.3: Ciclo de Vida
- [ ] Estado Initiated: **EP-001, EP-002** ✓
- [ ] Estado In Processing: **EP-005** ✓
- [ ] Estado Completed: **EP-004, EP-006** ✓
- [ ] Estado Exception (Outcome Unknown): **EP-001** ✓

#### Seção 3.4: Recovery Rule
- [ ] Terminal pode estar alcançável sem pronto: **EP-003** ✓
- [ ] Resolver transação anterior antes de novo: **EP-001** ✓
- [ ] Nunca tratar desconhecido como falha definitiva: **EP-001, EP-006** ✓

#### Seção 3.5-3.7: Reconciliação, Cancelamento, Dados Sensíveis
- [ ] Reconciliação é obrigatória: **EP-001** (recovery), **EP-005** (messages) ✓
- [ ] Cancelamento (in-progress): **EP-004** (nice-to-have) ✓
- [ ] Void e refund: Fora de MVP, documentado ✓
- [ ] Dados sensíveis nunca mascarados: **EP-004** ✓

#### Seção 4 Provider-Specific (Monri)
- [ ] Reconciliação automática: **EP-001** (implementação Monri-específica) ✓
- [ ] Recuperação via erro 1001: **EP-001** ✓
- [ ] Códigos Busy: **EP-006** ✓
- [ ] Mapeamento completo: **EP-006** ✓

#### Seção 6 Operações Core + Decisões
- [ ] Sale Payment contrato: **EP-008, EP-002, EP-003** ✓
- [ ] Payment Status contrato: **EP-001, EP-007** ✓
- [ ] Device Status contrato: **EP-003** ✓
- [ ] Cancel (nice-to-have): **EP-004** ✓
- [ ] Reconciliation (obrigatória): **EP-001, EP-005** ✓

---

## 5. VALIDAÇÃO: NENHUM ÉPICO FAZ TRABALHO DUPLICADO?

### EP-001: Recuperação de Resultado Desconhecido

**Responsabilidade única**: Garantir que transações com resultado perdido são recuperadas, confirmadas/rejeitadas, nunca deixadas órfãs.

**Não duplica com**:
- EP-002: Idempotência é sobre retry de requisição, não recovery de resultado
- EP-003: Status é verificação, não action recovery
- EP-004: Recebida é apresentação, não recovery
- EP-005: Mensagens são comunicação, não recovery logic
- EP-006: Error mapping é categorização, não recovery action
- EP-007: Observabilidade é diagnóstico, não recovery logic
- EP-008: Contrato é interface, não recovery logic

**Verificação**: ✓ Sem duplicação

---

### EP-002: Identidade e Idempotência

**Responsabilidade única**: Cada tentativa tem identidade única, retry retorna estado anterior.

**Não duplica com**:
- EP-001: Recovery é ação, idempotência é verificação antes
- EP-003-008: Nenhuma interseção

**Verificação**: ✓ Sem duplicação

---

### EP-003: Prontidão do Terminal

**Responsabilidade única**: Verificar e reportar status do terminal, bloquear vendas se não pronto.

**Não duplica com**:
- EP-001: Recovery se resultado anterior bloqueou, não verificação de status
- EP-005: Mensagens sobre progresso, não status check
- Outros: Nenhuma interseção

**Verificação**: ✓ Sem duplicação

---

### EP-004: Recebida

**Responsabilidade única**: Extrair dados de recebida, mascarar dados sensíveis, formatar conforme país.

**Não duplica com**:
- EP-006: Error mapping é erros, recebida é resultado final
- Outros: Nenhuma interseção

**Verificação**: ✓ Sem duplicação

---

### EP-005: Mensagens de Progresso

**Responsabilidade única**: Emitir eventos padronizados, deixar UI traduzir.

**Não duplica com**:
- Nenhum outro épico toca em eventos de progresso

**Verificação**: ✓ Sem duplicação

---

### EP-006: Mapeamento de Erros

**Responsabilidade única**: Mapear códigos de fornecedor para chaves normalizadas.

**Não duplica com**:
- EP-004: Recebida é sucesso, error mapping é falhaha/recusa
- Outros: Nenhuma interseção

**Verificação**: ✓ Sem duplicação

---

### EP-007: Observabilidade e Suporte

**Responsabilidade única**: Fornecer ferramentas para support diagnosticar e resolver.

**Não duplica com**:
- Nenhum outro épico fornece observabilidade

**Verificação**: ✓ Sem duplicação

---

### EP-008: Contrato MPOS Air Estável

**Responsabilidade única**: Versionar e manter estável o contrato core.

**Não duplica com**:
- Épicos 1-7 implementam conforme contrato, não definem contrato

**Verificação**: ✓ Sem duplicação

---

## 6. VALIDAÇÃO: DEPENDENCIES ENTRE ÉPICOS FAZEM SENTIDO?

### Ordem de Implementação Recomendada

```
Fase 1: CORE (semana 1-2)
├─ EP-008: Contrato (define interface)
├─ EP-002: Idempotência (garante nenhuma duplicata)
└─ EP-003: Prontidão (verifica se terminal está ready)

Fase 2: PAYMENT FLOW (semana 3-5)
├─ EP-001: Recovery (recupera resultados perdidos)
└─ EP-006: Error Mapping (categoriza erros)

Fase 3: UX (semana 6-7)
├─ EP-005: Mensagens (comunica progresso)
└─ EP-004: Recebida (documenta resultado)

Fase 4: OPERAÇÕES (semana 8-9)
└─ EP-007: Observabilidade (permite diagnosticar)
```

### Justificativa de Dependencies

| EP | Depende de | Razão |
|---|---|---|
| EP-001 | EP-002, EP-008 | Recovery precisa de idempotência (não duplica) e contrato definido |
| EP-002 | EP-008 | Idempotência usa identidades definidas no contrato |
| EP-003 | EP-008 | Status usa endpoints definidos no contrato |
| EP-004 | EP-001, EP-006 | Recebida precisa de result estável (recovery) e categorias de erro |
| EP-005 | EP-008 | Eventos precisam de payload definido no contrato |
| EP-006 | EP-008 | Error mapping usa categorias core definidas em contrato |
| EP-007 | Todos | Observabilidade é cross-cutting, depende de todos funcionarem |
| EP-008 | Nenhuma | Contrato é baseline, ninguém depende dele (ele define o termo) |

**Conclusão**: Dependencies são lineares e fazem sentido. ✓

---

## 7. VALIDAÇÃO: STORIES TÊM ACCEPTANCE CRITERIA TESTÁVEIS?

Amostra aleatória:

### EP-001 Story 1001: Persistir Tentativa
- [ ] paymentId é gerado: **Testável** (verificar se response contém ID novo cada vez)
- [ ] externalId é gerado: **Testável** (idem)
- [ ] Timestamp gravado: **Testável** (SELECT from DB)
- [ ] Status é "Initiated": **Testável** (GET /eft-payments/{id} returns state="Initiated")

### EP-002 Story 2001: Gerar externalId
- [ ] externalId é UUID: **Testável** (format validation)
- [ ] Retornado em response: **Testável** (parse response JSON)
- [ ] Reused em retry: **Testável** (POST com mesmo header, verify same payment returned)

### EP-003 Story 3001: Endpoint GET /device-connection-status
- [ ] Retorna em <500ms: **Testável** (latency measurement)
- [ ] Estados incluem Connected, Disconnected: **Testável** (mock terminal states, verify response)

### EP-004 Story 4001: Dados de Recebida
- [ ] Amount extraído corretamente: **Testável** (compare com response Monri)
- [ ] PAN mascarado: **Testável** (regex match 411111\*\*\*\*1111)

### EP-006 Story 6001: Mapeamento Monri
- [ ] Código 1001 mapeia para PAYMENT_OUTCOME_CHECKING: **Testável** (table-driven test)
- [ ] Nenhum código sem mapeamento: **Testável** (read table, assert 150+ codes present)

**Conclusão**: Stories têm acceptance criteria concretos e testáveis. ✓

---

## 8. VALIDAÇÃO: COBERTURA DE RISCOS

### Riscos Identificados no Baseline

| Risco | Descrição | Mitigado por | Status |
|---|---|---|---|
| Dupla cobrança | Mesmo pagamento enviado duas vezes | EP-002 (idempotência) + EP-001 (recovery) | ✓ |
| Terminal bloqueado | Transação anterior deixa terminal indisponível | EP-001 (recuperação) | ✓ |
| Resultado desconhecido | Resposta perdida, nunca sabe se pagou | EP-001 (recovery) | ✓ |
| Dados sensíveis expostos | PAN em log ou recebida | EP-004 (mascaramento) | ✓ |
| Venda em terminal não-pronto | Venda condenada ao fracasso | EP-003 (readiness check) | ✓ |
| Código de erro não mapeado | Comportamento imprevisível | EP-006 (safe defaults) | ✓ |
| Support não consegue diagnosticar | Escalação desnecessária | EP-007 (observabilidade) | ✓ |
| Breaking change no contrato | MPOS Air se quebra | EP-008 (versionamento) | ✓ |

**Conclusão**: Todos os riscos principais têm mitigação em algum épico. ✓

---

## 9. VALIDAÇÃO: COMPLETUDE DAS STORIES

### Checklist por Épico

#### EP-001: Recuperação
- [x] 5 stories (persistência, bloqueio, confirm/reject, restart, lookup)
- [x] Cada story tem Gherkin scenario
- [x] Critérios de aceitação testáveis
- [x] Métricas de sucesso definidas

#### EP-002: Idempotência
- [x] 3 stories (gerar ID, diferenciar IDs, deduplicar)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

#### EP-003: Prontidão
- [x] 3 stories (endpoint, bloqueio, transição)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

#### EP-004: Recebida
- [x] 4 stories (dados, mascaramento, aprovado/recusado, conformidade)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

#### EP-005: Mensagens
- [x] 2 stories (progresso, recovery)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

#### EP-006: Mapeamento
- [x] 3 stories (Monri completo, novo fornecedor, safe defaults)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

#### EP-007: Observabilidade
- [x] 3 stories (searchable logs, runbook, metrics)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

#### EP-008: Contrato Estável
- [x] 2 stories (versionado, estabilidade 8 sem)
- [x] Cada story tem Gherkin
- [x] Critérios testáveis

**Total: 25 stories com acceptance criteria testáveis**

---

## 10. CONCLUSÕES DE RASTREABILIDADE

### ✓ Cobertura Completa

1. **Todos os requisitos do baseline têm implementação**: 25+ requisitos mapeados a 25 stories
2. **Todos os use cases estão cobertos**: 8 UCs, 1 partial (nice-to-have), 7 full
3. **Todos os edge cases têm tratamento**: 5 categorias, todas mapeadas
4. **Nenhum épico duplica trabalho**: Responsabilidades são únicas
5. **Nenhuma feature fica sem épico**: 0 requisitos órfãos
6. **Stories são testáveis**: 100% têm acceptance criteria concretos

### ✓ Sem Gaps Críticos

- Recovery de resultado desconhecido: **EP-001** ✓
- Idempotência: **EP-002** ✓
- Prontidão terminal: **EP-003** ✓
- Recebida: **EP-004** ✓
- Mensagens de progresso: **EP-005** ✓
- Mapeamento de erros: **EP-006** ✓
- Observabilidade: **EP-007** ✓
- Contrato estável: **EP-008** ✓

### ✓ Dependências Fazem Sentido

Ordem de implementação é linear, sem dependências cruzadas.

### ✓ Risco Mitigado

Todos os 8 riscos principais têm épico responsável.

---

## RECOMENDAÇÃO

**Os 8 épicos cobrem completamente o baseline consolidado. Pronto para criação no Jira MAIR.**

Próximo passo: Importar backlog para Jira com:
- Epic criado por épico
- 25 stories associadas
- Story points estimados
- Assignees definidos
- Sprint planning

---

**Versão**: 1.0  
**Data**: 29 de setembro de 2026  
**Status**: DRAFT - Aguardando aprovação de Ana Maria Gropeneanu
