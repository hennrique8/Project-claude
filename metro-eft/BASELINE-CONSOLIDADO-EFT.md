---
document_type: "Baseline Funcional Consolidado"
project: "Metro EFT - M.POS Air Acceleration"
version: "1.0"
date_created: "2026-09-29"
author: "Claude Code - Análise Funcional"
scope: "Consolidação de documentação Confluence e identificação de contradições"
approver_required: "Ana Maria Gropeneanu"
reviewers_assigned: ["Markéta Janáková", "Payment Squad Metro", "Integration Architect"]
status: "DRAFT - Aguardando aprovação"
---

# BASELINE CONSOLIDADO: EFT Device Service para M.POS Air

## 1. PROPÓSITO E ESCOPO

Este documento consolida a documentação funcional dispersa em Confluence numa referência única para implementação. Segue a estrutura proposta no DRAFT do "EFT Functional Baseline, template and first content v0.1" (Flávio Oliveira, 22 Sep 2026), separando rigorosamente:

- **[BASELINE]**: Válido para todos os países e fornecedores
- **[PROVIDER-SPECIFIC: nome]**: Específico de um fornecedor
- **[COUNTRY-SPECIFIC: código]**: Específico de um país
- **[COUNTRY-SEMANTICS: código]**: Existe em todos mas significa diferente por país

Fontes primárias: Confluence espaço POS, últimas atualizações até 28 de setembro de 2026.

### Documentos consolidados neste baseline

| Documento | ID Confluence | Versão | Status | Data |
|-----------|---------------|--------|--------|------|
| EFT Integration Scope Definition | 2297987361 | 15 | Pendente aprovação | 28 Sep 2026 |
| Monri → MPOS Air Payment States & Error Mapping | 2618818586 | 11 | In review | 24 Sep 2026 |
| EFT Functional Baseline, template and first content | 2902491400 | 3 | In review | 22 Sep 2026 |
| EFT document version and approval standard | 2903212217 | 5 | In review | 22 Sep 2026 |
| Monri Sale Payment — Edge Cases and Recovery | 2618621992 | 3 | Draft | 27 Jul 2026 |

---

## 2. REGRAS DE GOVERNANÇA CRÍTICAS

### 2.1 Regra de Separação Funcional/Técnica

O BA funcional não duplica documentação técnica. Fronteira acordada:

- **Funcional**: O que o caixa vê e faz, resultados finais, ações permitidas, regras de país, decisões de negócio faltantes
- **Técnico**: Contratos de API, máquina de estados no código, mappers, protocolo campo a campo, persistência, logging

Quando o funcional precisa de facto técnico, referencia a página técnica do arquiteto. Não copia.

**Referências técnicas Core (Level 1-2):**
- 01-Overview: Definições de vocabulário e contexto
- 02-Core Concepts: Conceitos reutilizáveis
- 03-Payment State Machine: Máquina de estados completa
- 05-Error Mapping: Categorias de erro normalizadas
- 08-Error Framework: Framework de tratamento

### 2.2 Regra ADR-04: Vendor Adapters Must Not Contain Business Logic

Vendedor-específico = apenas mapeamento técnico (códigos, timeouts, endpoints). Não contém lógica de negócio. Se um adaptador de fornecedor precisa de uma regra de negócio (ex: "recusa é resultado, não erro"), essa regra vem do baseline funcional.

### 2.3 Governance Standard (Proposto por Flávio Oliveira)

Todo documento EFT funcional deve ter:

**Bloco de aprovação obrigatório no rodapé:**

| Versão | Data | Autor | Revisado por | Aprovado por | Status |
|--------|------|-------|--------------|--------------|--------|
| 1.0 | [data] | [autor] | [revisores nomeados] | Ana Maria Gropeneanu | Approved |

**Valores de Status:**
- `Draft`: Sendo escrito, pode mudar sem aviso
- `In review`: Completo na visão do autor, revisores nomeados foram solicitados
- `Reviewed`: Revisores responderam, comentários resolvidos
- `Approved`: Referência para implementação, mudanças precisam nova versão
- `Superseded`: Substituído, nome do sucessor obrigatório
- `Living`: Log/tracker atualizado continuamente, sem ciclo de aprovação

**Regra crítica**: "Uma página com status `Draft` não é base válida para acceptance criteria de uma story. Apenas páginas `Approved` são."

---

## 3. BASELINE FUNCIONAL: VÁLIDO PARA TODOS OS PAÍSES E FORNECEDORES

### 3.1 O que o EFT Device Service Entrega

O adaptador EFT existe para que o caixa nunca precise conhecer detalhes técnicos de nenhum fornecedor de pagamento. MPOS Air sempre fala a mesma linguagem de negócio; a equipe local faz essa linguagem funcionar com o fornecedor que cada país escolhe.

**Produto**: EFT Device Service e seu contrato com MPOS Air, entregue para todos os países. Uma capacidade do produto existe no serviço para todos os países; se está integrada com um fornecedor específico é decidido por país.

### 3.2 Operações Core - Contrato com MPOS Air

O contrato core entre MPOS Air e EFT Device Service cobre:

1. **Device Status** - Verificar se terminal está pronto
2. **Device Initialization** - Preparar terminal para pagamentos
3. **Sale Payment** - Executar pagamento de venda
4. **Cancellation (in-progress)** - Abortar pagamento em andamento (pedido do caixa)
5. **Payment Inquiry/Status** - Verificar status de pagamento anterior
6. **Reconciliation Hook** - Disparador para fechamento do dia

Implementações de fornecedor podem variar em capacidade, mas **nunca requerem mudanças no contrato MPOS Air**.

**Nota crítica**: Cancelamento feito pelo cliente no terminal EFT **não é uma operação do contrato**. É um resultado: o fornecedor retorna como outcome `cancelled` e MPOS Air recebe um resultado `declined`.

### 3.3 Ciclo de Vida de Pagamento - Visão de Negócio

**Estados Obrigatórios (mínimo):**

```
Initiated
  ↓
In Processing (terminal ativo com cliente)
  ↓
Completed (success/declined)

Exception State:
Outcome Unknown (comunicação caiu, resultado não confirmado)
```

**Regra de recuperação**: Um terminal pode estar alcançável sem estar pronto. O EFT Device Service **deve resolver a transação anterior** antes de reportar o terminal como pronto ou aceitar uma nova venda.

**Regra crítica**: Um resultado desconhecido **nunca é tratado como falha definitiva** enquanto o resultado real não for recuperado do fornecedor. Risco de dupla cobrança se o cliente tentar novamente.

### 3.4 Clarificação: Cancelamento vs. Void vs. Refund

Essas três operações são **diferentes e frequentemente confundidas**:

- **Cancelamento**: Aborda um pagamento **em andamento**, antes de resultado final
  - Iniciado: pelo caixa (pedido para parar) ou cliente (cancela no terminal)
- **Void**: Anula uma transação já **aprovada**, antes do settlement
  - Transação foi autorizada mas pode ser anulada antes de fechar o dia
- **Refund**: Devolve valor de transação **completed ou settled**, ou fluxo de devolução
  - Depois de finalizado, cliente quer reembolso

**Classificação MVP (acordada 22 Sep 2026)**:
- Cancelamento feito pelo cliente no terminal: sempre suportado (o fornecedor retorna como resultado)
- Cancelamento pedido pelo caixa via MPOS Air: nice-to-have, não é MVP
- Void: nice-to-have, não é MVP
- Refund: nice-to-have, não é MVP

**Tratamento**: Essas capacidades são **não parte do MVP mas consideradas na arquitetura** para integração futura sem fechar portas. Um país pode elevar uma delas a must-have se houver requisito legal ou do fornecedor. A necessidade é descoberta nos workshops Payment Squad com cada país e adicionada ao backlog se necessário.

### 3.5 Reconciliação (Fechamento do Dia)

**[BASELINE]** Reconciliação é uma capacidade de produto **obrigatória**. Todos os países devem fechar o dia.

**Como é disparada**: Depende do fornecedor e é registrado no baseline funcional específico do país.

Ver página dedicada de regra de reconciliação para detalhes de lógica de negócio.

### 3.6 Dados Sensíveis

Dados de cartão **nunca são armazenados ou registrados sem máscara**, para nenhum fornecedor. Track data, PIN block e cryptogram **nunca são persistidos ou retornados**.

### 3.7 Fora do Adaptador de Fornecedor (para todos)

O adaptador de fornecedor:
- **Não contém lógica de negócio**
- **Não realiza persistência de longo prazo**
- **Não decide a experiência do caixa**
- **Não trata de fiscalização** (responsabilidade de serviços dedicados)

---

## 4. PROVIDER-SPECIFIC: MONRI (CROATIA PILOT)

### 4.1 Reconciliação Automática

**[PROVIDER-SPECIFIC: Monri]**

Monri fecha o dia automaticamente em horário programado. Se o fechamento falha, o terminal retenta antes da próxima solicitação de transação. ECRs (terminais) não inicializam não monitorados em modo poupança de energia podem perder o fechamento programado.

**Consequência de backlog**: Nenhum disparador manual de reconciliação é necessário para o pilot Croatia.

**Fonte**: Monri Croatia EFT integration specification, seção 8.20: "ECR is not required to issue this command and we don't recommend it."

### 4.2 Recuperação de Resultado Desconhecido

**[PROVIDER-SPECIFIC: Monri]**

Monri protege contra duplicatas bloqueando o terminal. Se uma resposta de transação foi enviada mas a confirmação não chegou, o terminal retorna erro `1001` para qualquer nova requisição, inclusive uma nova venda.

O erro carrega dados da transação pendente, incluindo cópia da resposta original, permitindo ao serviço resolvê-la sem repetir a venda.

**Máquina de estados Monri**:
- Erro `1001`: "Waiting confirm" - recovery signal, não erro
- Erro `1002`: "Waiting voice authorization" (referral) - recovery signal
- Erro `1018`: "Waiting completion transaction" - recovery signal

Esses códigos indicam que há transação anterior que precisa ser resolvida antes de novo pagamento.

**Fonte**: Monri specification, seções 7.1 e 8.8.

### 4.3 Estados Ocupado (Busy)

**[PROVIDER-SPECIFIC: Monri]**

Monri retorna múltiplos códigos de ocupado, cada um com nuance técnica:

| Código | Significado técnico | Categoria de arquitetura |
|--------|-------------------|-------------------------|
| 2 | Device upgrade in progress | Busy |
| 4 | Terminal is busy | Busy |
| 1006 | Processing another transaction | Busy |
| 1007 | Processing confirm command | Busy |

**Pergunta de decisão pendente**: Esses codes merecem UI diferente, ou permanecem uma categoria "terminal ocupado" para o caixa?

**Fonte**: Monri error mapping table, linhas 2006, 2004, 1006, 1007. Técnico em 05-Error Mapping documento.

### 4.4 Mapeamento de Códigos Monri → Chaves Normalizadas MPOS Air

**Estrutura de 4 colunas (apenas primeira muda por fornecedor):**

```
Código Monri → Categoria arquitetura → Chave normalizada → Mensagem unificada (MPOS Air trata)
```

**Chaves normalizadas obrigatórias** (definidas no Core Framework):
- `TERMINAL_NOT_READY` - Terminal não pode iniciar venda
- `TERMINAL_BUSY` - Terminal processando outra operação
- `PAYMENT_OUTCOME_CHECKING` - Resultado sendo verificado, não retry cego
- `PAYMENT_REQUEST_INVALID` - Validação de entrada falhou
- `PAYMENT_DECLINED` - Recusa de negócio (cartão expirado, saldo insuficiente, etc)
- `PAYMENT_CANCELLED` - Cliente cancelou no terminal
- `PAYMENT_TECHNICAL_ERROR` - Erro não mapeado ou técnico

**Mapeamento Monri completo**: Ver seção 6 do "Monri → MPOS Air Payment States & Error Mapping" (2618818586), linhas de código 1 a 5006. Total: 150+ mapeamentos de Monri para 7 chaves.

**Regras de mapeamento (são constantes, não mudam):**
1. Recusa é resultado de negócio final, não erro técnico
2. Resultado desconhecido após despacho exige recuperação: nunca dizer falhou, nunca repetir às cegas
3. Retentabilidade é decidida pelo serviço, nunca inferida do POS
4. Códigos nativos Monri ficam dentro do serviço como diagnóstico protegido
5. Entrada desconhecida falha em segurança: vai para recuperação até ser mapeada propositalmente

---

## 5. COUNTRY-SPECIFIC E COUNTRY-SEMANTICS

### 5.1 Pending: Decisões Hungary e Romania

**[COUNTRY-SPECIFIC: HUN, ROU]**

Decisões equivalentes ainda pendentes de documentação de fornecedor. Esperando confirmação de:
- Qual fornecedor para cada país
- Capacidades suportadas
- Mapeamento de erros de fornecedor
- Regra de reconciliação (manual vs automático)

### 5.2 Licença Restringe Métodos de Pagamento

**[COUNTRY-SPECIFIC: HUN]**

Quando uma licença de cliente é usada na Hungria, dinheiro não é permitido e o caixa recebe aviso pop-up específico do país.

**Consequência EFT**: Se dinheiro está bloqueado e o pagamento por cartão falha, não há fallback. Terminal readiness se torna um bloqueador de venda nesse caso.

**Decisão de negócio faltante**: Qual é o comportamento esperado quando o cliente quer pagar por dinheiro mas está bloqueado, e cartão também falha?

**Fonte**: Cross-squad demo (15 Sep 2026).

### 5.3 Vocabulário de Status do Terminal

**[COUNTRY-SEMANTICS: Device Readiness]**

Terminal readiness tem significados diferentes por país/fornecedor:

| País/Fornecedor | Vocabulário | Significado |
|-----------------|------------|------------|
| Monri/Croatia | `Connected`, `Disconnected`, `NeedInitialization`, `Initializing` | Múltiplos estados |
| Portugal (actual) | Strictly binary: `connected` ou `not connected` | Apenas dois |
| Spain (actual) | Strictly binary: `connected` OU initializado corretamente | Inicialización implícita em "connected" |

**Baseline recomendação**: Manter flexibility, supplier-specific. Uma tabela de gap por país define qual vocabulário e triggers são usados localmente.

**Fonte**: Scope Definition UC-03; Edge Cases seção 4; experiência Spain/Portugal material.

### 5.4 Recebida (Regra de Impressão)

**[COUNTRY-SPECIFIC: ROU, HUN, HRV, ESP, POR]**

Regra legal local, cópias exigidas, proprietário do template, regra de falha de impressão.

**Nunca herdar de outro país por analogia**: As regras de cópia divergem entre países vizinhos.

**Working assumption acordada (22 Sep 2026)**:
- EFT Device Service encaminha o payload de resultado de pagamento para MPOS Air
- MPOS Air decide o que é impresso no recebida
- Regra de país é aplicada em MPOS Air, não no EFT Device Service
- A ser confirmado nos workshops de recebida com cada país

**Fonte**: EFT Receipt baseline vs. Monri Fit (2618753077); EFT Receipt Spain and Portugal Baselines (2769027185).

---

## 6. OPERAÇÕES CORE E DECISÕES PENDENTES

### 6.1 Sale Payment (UC-01)

**Contrato existente (MPOS Air → EFT Device Service):**

```
POST /eft-payments

Request body obrigatório:
- paymentOrderId: ID de pedido no MPOS Air
- paymentLineId: ID de linha de pagamento
- storeNumber: Loja
- tillNumber: Caixa (resolve terminal + config)
- userId: Operador
- amount: Valor em centavos
```

**Decisões pendentes de confirmação:**
- Incluir `currency` no contrato? (Monri não exige, recomendado para validação neutro)
- Response schema exato (imediato vs. via SignalR)
- Qual identificador de retry é estável e reenviado no transporte (externalId ou header de idempotência)?

**Fonte**: Scope Definition UC-01; Working Baseline seção 3-4.

### 6.2 Payment Status/Inquiry (UC-02)

**Contrato existente (MPOS Air → EFT Device Service):**

```
GET /eft-payments/{id}

Response:
- id: EFT payment ID (service-owned)
- state: Initiated, EFTProcessing, Completed, Failed
- error.code: Chave normalizada (ex: PAYMENT_DECLINED)
- error.additionalInformation: Dados de diagnóstico
```

**Implicação crítica**: Status vem de estado durável do serviço, nunca bloqueia em operação longa do terminal.

**Decisões pendentes**: Como MPOS Air detecta/sonda quando uma transação em recovery foi resolvida?

### 6.3 Device Connection Status (UC-03)

**Contrato existente (MPOS Air → EFT Device Service):**

```
GET /device-connection-status

Response states (variable por supplier):
- Monri: Connected, Disconnected, NeedInitialization, Initializing
- Portugal actual: connected or not connected
- (etc, definir por país)
```

**Decisão crítica pendente**: Qual é o vocabulário de pronto vs. ocupado vs. bloqueado? Unificar ou supplier-specific?

### 6.4 Cancel In-Progress Payment (UC-04)

**[BASELINE] Classificação**: Nice-to-have, não MVP.

Cancelamento **pedido pelo caixa** via MPOS Air: não integrado com fornecedor para o pilot. Considerado na arquitetura.

Cancelamento **feito pelo cliente no terminal**: sempre suportado, porque o fornecedor retorna como outcome `user_cancelled`.

**Nota de decisão (22 Sep 2026)**: Product capability, considerado na arquitetura. Se um país precisa de cancelamento pedido pelo caixa, descobre nos workshops Payment Squad e adiciona ao backlog.

**Método proposto (não MVP):** SignalR hub method `Revert`.

### 6.5 Reconciliation/Settlement (UC-05)

**[BASELINE] Reconciliação é obrigatória.** Todos os países fecham o dia.

**Como é disparada**: Depende do fornecedor (automático vs. manual).

- Monri: Automático, nenhum trigger manual necessário para Croatia
- Hungary/Romania: Aguardando fornecedor

**Método proposto:**
```
POST /reconciliation

Header obrigatório:
- x-action-id: Identificador idempotência (reconciliation é idempotente)
```

**Consequência downstream**: Gera Z-report de transação via User-Service/Transaction Service após reconciliação bem-sucedida com middleware/banco.

### 6.6 Initialization/Send Pending (UC-06)

**Capabilidade de produto**: Sim.

**Método proposto:**
```
POST /send-pending-transactions

Orquestra fluxo de inicialização ou transações pendentes.
```

**Decisões pendentes**: Quando é disparado? Manual ou automático?

### 6.7 Reset Device (UC-07)

**Capabilidade de produto**: Sim.

**Método proposto:**
```
GET /reset-device

Dispara reset/unblock da integração EFT/dispositivo.
```

### 6.8 Refund (UC-08)

**[BASELINE] Classificação**: Nice-to-have, não MVP.

Refund é uma capabilidade de produto considerada na arquitetura. Não integrada para o pilot Croatia. Cada país descobre a necessidade nos workshops Payment Squad.

Se um país precisa (requisito legal ou fornecedor), adiciona ao backlog.

---

## 7. CONTRADIÇÕES IDENTIFICADAS E RESOLUÇÕES

### Contradição 1: Escopo de Refund

**Observação**: Três documentos definem diferente.

| Documento | Definição |
|-----------|-----------|
| Scope Definition UC-08 (v15, 28 Sep) | Nice-to-have, considerado em arquitetura |
| Baseline Funcional DRAFT (v3, 22 Sep) | Nice-to-have, não MVP, considerado em arquitetura |
| Edge Cases (v3, 27 Jul) | Não mencionado (documento anterior) |

**Resolução**: Consenso foi alcançado em 22 Sep 2026. Todos os documentos posteriores concordam com "nice-to-have, não MVP, considerado em arquitetura". Baseline atual está **correto**.

---

### Contradição 2: Reconciliation Manual vs. Automático

**Observação**: Scope Definition diz mandatory, Edge Cases pede decisão.

| Documento | Definição |
|-----------|-----------|
| Scope Definition (nota 22 Sep) | Reconciliation é mandatory product capability |
| Baseline Funcional (seção 2.5) | Reconciliation é mandatory product capability |
| Monri-specific (seção 3.1) | Monri fecha automaticamente, nenhum trigger manual |
| Edge Cases (não tem seção) | Não cobre reconciliation |

**Resolução**: Capacidade é obrigatória (todos os países fecham), **mas como é disparada depende do fornecedor**:
- Monri: Automático (Croatia)
- Hungary/Romania: Aguardando fornecedor

Baseline está **correto**. Gap está documentado e esperando decisão de fornecedor.

---

### Contradição 3: Terminologia - Cancellation vs. Code 11

**Observação**: Monri specification e documentação MPOS Air usam "cancellation" genericamente.

| Documento | Definição |
|-----------|-----------|
| Baseline Funcional (seção 2.6) | Cancelamento (in-progress) vs. Void (approved) vs. Refund (completed) |
| Monri error code 11 (seção 4.2) | "Transaction aborted due to ECR abort command" |
| Monri error codes 1030, 1040, 1100, 1203, 2003, 3002, 3004, 3009 | "User canceled" (cliente no terminal) |

**Resolução**: Código 11 é **ECR (terminal) enviou comando abort** (técnico). Cancelamentos de cliente usam códigos diferentes. **Precisão Monri**: Código 11 não é "cancelamento de cliente genérico"; é comunicação do caixa ao terminal. Documentação atual está **correto**.

---

### Contradição 4: Terminal Status Vocabulary

**Observação**: Scope Definition lista estados, Edge Cases diz Spain/Portugal é binary.

| Documento | Definição |
|-----------|-----------|
| Scope Definition UC-03 | Connected, Disconnected, NeedInitialization, Initializing |
| Edge Cases seção 4 | Spain/Portugal: "strictly binary - connected or not connected" |
| Edge Cases seção 4 | Spain: connected implica initialization foi bem-sucedida |
| Monri-specific | Múltiplos estados suportados |

**Resolução**: **Não é contradição**, é variação por supplier e país. Baseline recomenda flexibility. Cada país define seu vocabulário numa tabela de gap. Documentação está **correto**.

---

### Contradição 5: Error 1001 vs. Code 11 Confusão

**Observação**: Duas situações completamente diferentes estavam sendo confundidas.

| Código | Significado | Tipo |
|--------|-----------|------|
| 1001 | "Waiting confirm" - transação anterior pendendo confirmação | Recovery signal - não é erro |
| 11 | "Transaction aborted due to ECR abort command" - caixa enviou abort | User cancellation |
| 1002 | "Waiting voice authorization" - autorização por voz pendente | Recovery signal |
| 1018 | "Waiting completion transaction" | Recovery signal |

**Resolução**: Documentação corrigiu essa confusão. Monri specification seção 8.8 e 8.29 esclarecem. Baseline está **correto**. Qualquer material que usa 11 como "cliente cancela" está **errado**.

---

## 8. GAPS EXPLÍCITOS E ABERTOS

### Gap 1: Decisões Hungary e Romania

**O que falta**: Documentação de fornecedor, scope funcional, mapeamento de erros, regra de reconciliação.

**Consequência**: Não há baseline country-specific ainda. Será criado quando fornecedor for escolhido.

**Owner**: Metro business (escolher fornecedor), CGI (criar documentação).

### Gap 2: Busy Codes Merecem UI Diferente?

**O que falta**: Decisão se Monri codes 2, 4, 1006, 1007 merecem comportamento de UI diferente.

**Consequência**: Hoje todos mapeiam para `TERMINAL_BUSY`, mensagem única. Poderia ser refinado.

**Owner**: Payment Squad (business), MPOS Air (UI).

**Fonte**: Edge Cases seção 4, ponto de decisão aberto.

### Gap 3: Politica de Retry e Idempotência Exata

**O que falta**: Qual é o identificador de retry estável (externalId vs. header idempotência)?

**Consequência**: Contrato não está 100% definido. Proposta existe (externalId service-owned), aguarda confirmação de MPOS Air.

**Owner**: MPOS Air Solution Architect, Integration Architect.

**Fonte**: Working Baseline seção 3, placeholder "Confirm exactly how MPOS Air resubmits".

### Gap 4: Confirm vs. Reject Rule

**O que falta**: Regra determinística exata para confirmar ou rejeitar após interrupção.

**Consequência**: Documentação propõe "confirmar apenas matching, durably retained response", mas regra exata precisa validação com Monri.

**Owner**: Integration Architect + Monri.

**Fonte**: Working Baseline seção 7.

### Gap 5: Operações Não-MVP Não Estão em Documentação Técnica Core

**O que falta**: Void, refund e cancelamento pedido pelo caixa não estão em nenhum documento de design técnico.

**Consequência**: Estão "considerados em arquitetura" mas sem detalhes. Quando um país elevar a must-have, precisará de design completo.

**Owner**: Integration Architect (design quando necessário).

**Fonte**: Baseline Funcional seção 2.6.

---

## 9. ÉPICOS OBRIGATÓRIOS (TODO PROJETO EFT)

Esses oito épicos devem estar no backlog de **todo projeto EFT**, independente de país ou fornecedor. Não podem ser omitidos.

### Épico 1: Recuperação de Resultado Desconhecido

**[BASELINE]** Obrigatório.

Resposta perdida após despacho, confirmação perdida, reinício do serviço com pagamento em curso, sinal de espera de confirmação do terminal. Recuperação pelo identificador do pagamento em vez de nova venda.

**User Story exemplo:**
- Como caixa, quando comunicação cai durante pagamento, quero recuperar resultado sem criar duplo pagamento, para não cobrar cliente duas vezes.
- Critério: Transação é identificada por paymentLineId/paymentId, estado é recuperado do terminal, terminal é bloqueado até resolução.

**Métrica de sucesso**: Taxa de pagamentos em recovery resolvidos <2h.

---

### Épico 2: Identidade e Idempotência

**[BASELINE]** Obrigatório.

Identificador único por tentativa, persistido antes da chamada ao fornecedor. Retry de transporte devolve o estado existente e nunca emite segunda venda.

**User Story exemplo:**
- Como serviço, quando POS retenta um pagamento, quero retornar estado anterior, para garantir que uma transação não é enviada duas vezes ao fornecedor.
- Critério: externalId é gerado, persistido, e retornado; retry com mesmo externalId retorna estado existente sem nova venda.

**Métrica de sucesso**: Zero duplicatas de transação detectadas.

---

### Épico 3: Prontidão do Terminal

**[BASELINE]** Obrigatório.

Vocabulário único de estados, bloqueio de início de venda com terminal indisponível, mensagem de ocupado distinta de falha.

**User Story exemplo:**
- Como caixa, quero saber se terminal está pronto antes de iniciar venda, para entender se devo tentar outra vez ou usar outro método de pagamento.
- Critério: GET /device-status retorna estados claros (Ready vs. Busy vs. NotReady), startup é bloqueado se estado não é Ready.

**Métrica de sucesso**: Caixa consegue identificar bloqueador em <30s.

---

### Épico 4: Recebida

**[BASELINE]** Obrigatório.

Dados normalizados, mascaramento do PAN incluindo cartões de fidelização, comportamento em falha de impressão, regra legal local.

**User Story exemplo:**
- Como caixa, quero recebida do EFT com dados corretos mascarados, para poder confiar no documento e não expor dados de cartão.
- Critério: PAN mascarado (ex: 4111****1111), valor correto, referência autorização, data/hora, timestamp, regra de cópia do país.

**Métrica de sucesso**: Recebidas impressas com 100% de conformidade legal.

---

### Épico 5: Mensagens de Progresso

**[BASELINE]** Obrigatório.

Serviço emite chaves de evento; POS é dono do texto localizado e aprovado.

**User Story exemplo:**
- Como caixa, quero ver mensagens de progresso claras ("Insira cartão", "PIN necessário", "Autorizando"), para entender o que o terminal está fazendo.
- Critério: EFT Service emite `PRESENT_CARD`, `PIN_REQUIRED`, `AUTHORIZING` etc. MPOS Air traduz para linguagem local aprovada.

**Métrica de sucesso**: Usuários entendem o progresso, taxa de "contato suporte" <5%.

---

### Épico 6: Mapeamento de Erros

**[BASELINE]** Obrigatório.

Única coluna que muda por fornecedor é o código nativo. Resto é reutilizável: categoria arquitetura, chave normalizada, mensagem unificada.

**User Story exemplo:**
- Como BA, quando novo fornecedor entra, quero apenas mapear seus códigos de erro para chaves existentes, para não duplicar documentação.
- Critério: Monri 2 → Busy → `TERMINAL_BUSY` → "Terminal processando outra operação". Novo fornecedor: apenas coluna 1 muda.

**Métrica de sucesso**: Novo mapeamento leva <4h (não <1 dia).

---

### Épico 7: Observabilidade e Suporte

**[BASELINE]** Obrigatório.

Procura por qualquer identificador (paymentId, externalId, terminalId), sequência de estados com carimbos temporais, runbook para terminal bloqueado, resultado desconhecido, falha de impressão.

**User Story exemplo:**
- Como support, quero procurar um pagamento e ver toda a sequência (timestamps, estados, tentativas), para diagnosticar problemas sem acessar código.
- Critério: Dashboard/tool permite procurar por paymentId, externalId, terminalId. Timeline mostra estados, transições, tentativas, recovery.

**Métrica de sucesso**: Suporte resolve 80% dos casos sem escalar ao dev.

---

### Épico 8: Contrato MPOS Air Estável

**[BASELINE]** Obrigatório.

Contrato entre MPOS Air e EFT Device Service é tratado como fixo. Mudanças são explícitas e raras.

**User Story exemplo:**
- Como arquiteto, quero que operações core (sale, status, readiness) permaneçam estáveis, para que múltiplos fornecedores funcionem sem mudanças no cliente.
- Critério: POST /eft-payments, GET /eft-payments/{id}, GET /device-connection-status permanecem contratuais.

**Métrica de sucesso**: Zero breaking changes em 8 meses de operação.

---

## 10. MATRIZ DE RASTREABILIDADE

| Épico | Uso Cases | Documentação Core | Documentação Monri | Stories | Status |
|-------|-----------|-------------------|-------------------|---------|--------|
| Recuperação de Resultado | UC-02 Status, UC-05 Reconciliation | 03-Payment State Machine, Recovery Rules | Seção 7 "Recovery rules", código 1001/1002/1018 | 5+ | Partial |
| Idempotência | UC-01 Sale | 02-Core Concepts | Seção 7 "Retry and idempotency" | 3+ | Partial |
| Prontidão Terminal | UC-03 Device Status | 03-Payment State Machine | Seção 2 "Lifecycle rules" | 4+ | Partial |
| Recebida | UC-01 Sale, UC-05 Reconciliation | Especificação país | Receipt baseline vs. Monri Fit | 3+ | Not started |
| Mensagens Progresso | UC-01 Sale | Vocabulário core | Seção 8 "Progress events" | 2+ | Partial |
| Mapeamento Erros | Todos | 08-Error Framework | Tabela completa Monri codes | 2+ | Partial |
| Observabilidade | Todos | Logging/traceability | Não coberto | 4+ | Partial |
| Contrato Estável | Todos | 05-Error Mapping | Seção 3-4 "Contract baseline" | 1+ | Done |

---

## 11. RECOMENDAÇÕES IMEDIATAS

### 11.1 Ações até fim de setembro 2026

**Semana 1 (29 Sep - 5 Oct):**
- [ ] Este baseline consolidado é revisado com Markéta Janáková e Ana Maria Gropeneanu
- [ ] Gaps 1-5 acima são formalmente escalados como "decisões pendentes" no RAID log
- [ ] Cada gap recebe um owner e data de resolução esperada

**Semana 2-3 (6-19 Oct):**
- [ ] Documentação Confluence é atualizada com bloco de aprovação padrão (proposto 18 Sep, ainda não aplicado)
- [ ] Scope Definition (v15) é trazido a aprovação após alinhamento com gaps
- [ ] Working Baseline Monri (v11) recebe data-alvo de aprovação

---

### 11.2 Pré-requisito para épicos

Antes de criar stories de épicos, **cada documento core deve estar Approved**:

| Documento | Aprovador | Revisor | Data alvo |
|-----------|-----------|---------|-----------|
| Baseline Funcional DRAFT (agora v1.0) | Ana Maria Gropeneanu | Markéta, Payment Squad | 15 Oct 2026 |
| Working Baseline Monri | Ana Maria Gropeneanu | Valentin, Cristian, Integration Architect | 15 Oct 2026 |
| Scope Definition | Markéta Janáková | Integration Architect | 15 Oct 2026 |
| Document Standard | Ana Maria Gropeneanu | Markéta Janáková | 15 Oct 2026 |

---

### 11.3 Próximo Passo: Backlog para Q1 2026

Com baseline aprovado, criar epics no Jira projeto MAIR com:
- Referência a página Confluence aprovada
- 5-7 user stories por épico (ver exemplos acima)
- Estimativa em story points
- Priorização por critério RICE
- Dependências entre épicos

---

## 12. REFERÊNCIAS E CITAÇÕES

**Documentação primária Confluence (POS space):**

| Página | ID | Versão | Data |
|--------|----|---------|----|
| EFT Integration Scope Definition | 2297987361 | 15 | 28 Sep 2026 |
| Monri → MPOS Air Payment States & Error Mapping | 2618818586 | 11 | 24 Sep 2026 |
| EFT Functional Baseline (DRAFT) | 2902491400 | 3 | 22 Sep 2026 |
| EFT document version and approval standard | 2903212217 | 5 | 22 Sep 2026 |
| Monri Sale Payment Edge Cases & Recovery | 2618621992 | 3 | 27 Jul 2026 |
| EFT Receipt: baseline vs. Monri Fit | 2618753077 | - | 26 Aug 2026 |
| EFT Receipt: Spain and Portugal Baselines | 2769027185 | - | 24 Aug 2026 |
| EFT: Squad level weekly tracker - Open Points | 2296906573 | - | 11 Sep 2026 |

**Documentação técnica core (referências, não reproduzidas):**
- 01-Overview: Contexto e vocabulário
- 02-Core Concepts: Conceitos reutilizáveis
- 03-Payment State Machine: Máquina de estados completa
- 05-Error Mapping: Categorias de erro normalizadas
- 08-Error Framework: Framework de tratamento

**Especificações externas:**
- Monri Croatia EFT integration specification (fornecedor)
- ADR-04: Vendor Adapters Must Not Contain Business Logic
- Governance padrão: METRO/CGI handover documentation

---

## HISTÓRIA DE APROVAÇÃO

| Versão | Data | Autor | Revisado por | Aprovado por | Status | Notas |
|--------|------|-------|---|---|---|---|
| 0.1 (DRAFT) | 29 Sep 2026 | Claude Code | Markéta Janáková | - | DRAFT | Consolidação inicial de 5 documentos Confluence |
| 1.0 | [pending] | Claude Code | Markéta Janáková, Payment Squad | Ana Maria Gropeneanu | In Review | Aguardando aprovação formal |

---

**Fim do baseline consolidado.**

Este documento substitui a leitura de 5 documentos Confluence espalhados. Mantém referências para detalhes técnicos sem duplicar. Serve como fonte de verdade funcional para implementação e criação de epics.

Próximo passo: Aprovação por Ana Maria Gropeneanu e Markéta Janáková, depois transformação em 8 épicos no Jira MAIR.
