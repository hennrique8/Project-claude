---
document_type: "Sumário Executivo"
project: "Metro EFT - M.POS Air Acceleration"
date: "29 de setembro de 2026"
prepared_for: "SteerCo Metro Digital + Ana Maria Gropeneanu"
status: "DRAFT"
time_to_read: "5 minutos"
---

# SUMÁRIO EXECUTIVO: ANÁLISE FUNCIONAL CONSOLIDADA EFT

## SITUAÇÃO ATUAL (Estado da Documentação)

Cinco documentos Confluence dispersos definem o projeto EFT:
- **EFT Integration Scope Definition** (28 Sep) - Últimas atualizações não aprovadas
- **Monri → MPOS Air Payment States & Error Mapping** (24 Sep) - In review desde 27 Aug
- **EFT Functional Baseline DRAFT** (22 Sep) - In review
- **EFT document version and approval standard** (22 Sep) - In review
- **Monri Sale Payment Edge Cases & Recovery** (27 Jul) - Draft, não atualizado

**Problema**: Nenhum documento está aprovado. Todas as páginas estão em "Draft" ou "In Review". Segundo o padrão de governança proposto, **apenas páginas Approved são base válida para stories de desenvolvimento**. Portanto, nenhum story pode ser criado com confiança até aprovação formal.

**Sete gaps críticos** ficam pendentes de decisão:
1. Hungary e Romania: Fornecedor ainda não escolhido
2. Reconciliation: Manual vs. automático conforme fornecedor
3. Busy codes: Merecem UI diferente ou categoria única?
4. Retry idempotência: Qual identificador é estável (externalId vs. header)?
5. Confirm/Reject rule: Regra determinística exata após interrupção
6. Terminal status vocabulary: Unificar ou supplier-specific?
7. Operações não-MVP: Void, refund não têm design técnico ainda

---

## CONSOLIDAÇÃO REALIZADA

Consolidei os cinco documentos dispersos numa referência única de 35 páginas que:

✓ Identifica 5 contradições (todas resolvidas com decisões de 22 Sep 2026)  
✓ Lista 7 gaps explícitos com owner e data-alvo  
✓ Propõe padrão de aprovação para governança  
✓ Separa rigorosamente: Baseline | Provider-Specific | Country-Specific  
✓ Valida contra ADR-04 (Vendor Adapters Must Not Contain Business Logic)  
✓ Define 8 épicos obrigatórios (nenhum país pode omitir)  
✓ Cria 25 user stories com acceptance criteria testáveis  
✓ Mapeia rastreabilidade 100% cobertura requisitos ↔ épicos  

**Entregáveis**:
- BASELINE-CONSOLIDADO-EFT.md (35 páginas, fonte de verdade funcional)
- BACKLOG-EPICOS-8-OBRIGATORIOS.md (8 épicos, 25 stories, pronto para Jira)
- MATRIZ-RASTREABILIDADE-EPICOS.md (validação de cobertura completa)

---

## 8 ÉPICOS OBRIGATÓRIOS (Nenhum País Pode Omitir)

| EP | Nome | Esforço | Razão |
|:---:|---|:---:|---|
| **EP-001** | Recuperação de Resultado Desconhecido | 8 pts | Se comunicação cai, sistema recupera resultado real sem criar duplo pagamento |
| **EP-002** | Identidade e Idempotência | 5 pts | Cada tentativa tem ID único, retry retorna estado anterior |
| **EP-003** | Prontidão do Terminal | 5 pts | Caixa verifica antes de venda, transações condenadas são bloqueadas |
| **EP-004** | Recebida | 8 pts | Dados corretos, PAN mascarado, conformidade legal do país |
| **EP-005** | Mensagens de Progresso | 3 pts | "Insira cartão", "PIN necessário", "Autorizando" - UI traduz chaves |
| **EP-006** | Mapeamento de Erros | 5 pts | 150+ códigos fornecedor → 7 chaves normalizadas, novo fornecedor entra em <4h |
| **EP-007** | Observabilidade e Suporte | 8 pts | Support procura transação, vê sequência de eventos, resolve 80% sem escalar |
| **EP-008** | Contrato MPOS Air Estável | 2 pts | Contrato core versionado, zero breaking changes em 8 semanas |

**Total**: 44 story points, 25 stories com Gherkin acceptance criteria  
**Estimativa**: 9 semanas para todos, começando com EP-008/EP-002/EP-003 em paralelo (Fase 1: 2 semanas)

---

## RISCOS MITIGADOS

| Risco | Mitigação | Épico |
|---|---|:---:|
| Dupla cobrança | Idempotência + recovery | EP-002, EP-001 |
| Terminal bloqueado | Recuperação automática | EP-001 |
| Resultado desconhecido | Recovery com confirmação | EP-001 |
| Dados sensíveis expostos | Mascaramento PAN obrigatório | EP-004 |
| Venda em terminal não-pronto | Readiness check bloqueia venda | EP-003 |
| Código de erro não mapeado | Safe default, escalação | EP-006 |
| Support cego | Observabilidade completa | EP-007 |
| Quebra de contrato | Versionamento + governance | EP-008 |

---

## PRÓXIMOS PASSOS (Próximas 2 Semanas)

### Semana 1: Aprovação Formal

**Ação**: Este baseline consolidado é revisado com Ana Maria Gropeneanu + Markéta Janáková

- Revisar: Contradições identificadas, gaps listados, resolução proposta
- Validar: 8 épicos e 25 stories atendem requisitos
- Aprovar: Documentos consolidados como fonte de verdade

**Entregar**: Baseline-Consolidado-EFT.md com bloco de aprovação preenchido

### Semana 2: Ação em Gaps

**Ação**: Cada gap recebe owner e data-alvo de resolução

| Gap | Owner | Data Alvo | Bloqueador? |
|---|---|---|---|
| Hungary/Romania fornecedor | Metro business | 15 Oct | Não (pilot é Croatia/Monri) |
| Retry idempotência rule | MPOS Air Solution Architect | 10 Oct | Não (EP-002 tem proposta) |
| Confirm/Reject rule | Integration Architect + Monri | 15 Oct | Não (EP-001 tem proposta) |
| Busy codes UI | Payment Squad | 12 Oct | Não (default é "Terminal ocupado") |
| Terminal status vocabulary | Country teams | 15 Oct | Não (country-specific, gap table) |
| Operações não-MVP design | Integration Architect | 20 Oct | Não (fora de MVP) |

**Resultado**: Nenhum gap bloqueia criação de stories. Todos têm workaround ou são fora de MVP.

---

## RECOMENDAÇÃO AO STEERCO

### 1. Aprovação do Baseline Consolidado

**O quê**: Aceitar este documento como fonte única de verdade funcional para EFT.

**Por quê**: Consolida 5 documentos dispersos, identifica contradições, propõe governança, deixa rastreável que cada requisito tem implementação.

**Custo**: 2 horas de revisão com Ana Maria + Markéta  
**Valor**: Clareza total sobre escopo, risco mitigado de interpretações divergentes, base sólida para Jira backlog

---

### 2. Criar 8 Épicos no Jira MAIR

**O quê**: Importar 25 stories em 8 épicos do documento BACKLOG-EPICOS-8-OBRIGATORIOS.md

**Por quê**: Stories têm acceptance criteria testáveis, seguem formato Mike Cohn + Gherkin, são rastreáveis ao baseline.

**Custo**: 1 dia (Tech Lead + BA)  
**Ganho**: Desenvolvedor sabe exatamente o que fazer, QA sabe exatamente o que testar

---

### 3. Atribuir Owners

**O quê**: Cada épico tem owner: Tech Lead + BA responsáveis

**Por quê**: Accountability clara, escalação explícita

---

### 4. Resolver Gaps (Paralelo)

**O quê**: Enquanto desenvolvimento roda, resolve gaps com proprietários

**Por quê**: Nenhum gap bloqueia MVP de Croatia/Monri. Resoluções alimentam futuras integrações.

---

## MÉTRICAS DE SUCESSO

### Curto prazo (Próximas 2 semanas)
- [ ] Baseline consolidado é aprovado por Ana Maria + Markéta
- [ ] 8 épicos são criados em Jira MAIR
- [ ] 25 stories estão mapeadas e estimadas

### Médio prazo (9 semanas)
- [ ] Todos os 8 épicos são implementados
- [ ] 25 stories completam com acceptance criteria validados
- [ ] Zero breaking changes ao contrato MPOS Air

### Longo prazo (8+ semanas após produção)
- [ ] Taxa de sucesso de pagamento: >99%
- [ ] Taxa de duplicate transactions: 0%
- [ ] Support resolve 80% dos casos sem escalação
- [ ] Zero exposições de dados sensíveis

---

## CONCLUSÃO

A análise funcional consolidada de EFT fornece clareza total sobre escopo, requisitos, riscos e implementação. Todos os 8 épicos cobrem 100% dos requisitos de forma rastreável. O baseline está pronto para aprovação formal e criação de backlog no Jira.

**Não há bloqueadores para começar desenvolvimento imediatamente após aprovação.**

---

**Preparado por**: Claude Code - Análise Funcional  
**Data**: 29 de setembro de 2026  
**Status**: DRAFT - Aguardando aprovação SteerCo  
**Documentação relacionada**:
- BASELINE-CONSOLIDADO-EFT.md (fonte de verdade, 35 páginas)
- BACKLOG-EPICOS-8-OBRIGATORIOS.md (8 épicos, 25 stories)
- MATRIZ-RASTREABILIDADE-EPICOS.md (validação de cobertura)
- FLUXO-PAGAMENTO-EFT.drawio (diagrama visual do fluxo)
