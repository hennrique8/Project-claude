---
document_type: "Backlog de Épicos"
project: "Metro EFT - M.POS Air Acceleration"
version: "1.0"
date_created: "2026-09-29"
author: "Claude Code - Análise Funcional"
status: "DRAFT - Pronto para Jira MAIR"
---

# BACKLOG: 8 ÉPICOS OBRIGATÓRIOS EFT

## Introdução

Esses oito épicos são obrigatórios em **todo projeto EFT**, independente de país ou fornecedor. Cada um tem user stories com critérios de aceitação testáveis. Seguem o formato Mike Cohn + Gherkin (As a / I want / So that + Given/When/Then).

Todos os épicos derivam do baseline consolidado no documento BASELINE-CONSOLIDADO-EFT.md.

---

## ÉPICO 1: RECUPERAÇÃO DE RESULTADO DESCONHECIDO

**ID**: EFT-EP-001  
**Prioridade**: Critical  
**Esforço estimado**: 8 pontos  
**Owner**: EFT Squad  

### Objetivo do Épico

Implementar recuperação de transações sem resultado confirmado. Quando comunicação cai durante pagamento (response perdida, confirmação perdida, reinício do serviço), o sistema recupera o resultado real do fornecedor sem criar duplo pagamento.

**Regra de negócio crítica**: Um resultado desconhecido **nunca é tratado como falha definitiva** enquanto o resultado real não for recuperado do fornecedor. Risco de dupla cobrança se cliente tentar novamente.

### User Stories

#### EFT-1001: Persistir Tentativa Antes de Contactar Fornecedor

**Como**: Desenvolver EFT Device Service  
**Quero**: Persistir a tentativa de pagamento em base de dados antes de qualquer chamada ao fornecedor  
**Para que**: Se o serviço falhar entre persistência e chamada, a tentativa não é perdida

**Cenário**: Transação perdida após persistência
- **Dado**: Uma requisição `POST /eft-payments` é recebida com paymentOrderId, paymentLineId, amount
- **E**: A tentativa é persistida em DB com estado "Initiated"
- **E**: O serviço falha antes de chamar Monri
- **Quando**: O serviço reinicia e verifica estado da tentativa
- **Então**: A tentativa é encontrada em DB com estado "Initiated"
- **E**: Nenhuma chamada duplicada foi enviada ao Monri

**Critérios de Aceitação**:
- paymentId é gerado, persistido e retornado imediatamente
- externalId é gerado, persistido, único por tentativa
- Timestamp de criação é gravado
- Terminal, valor, moeda, paymentLineId são persistidos
- Status é "Initiated" antes de qualquer atividade com fornecedor

---

#### EFT-1002: Recuperar Transação Anterior Bloqueada

**Como**: Caixa MPOS Air  
**Quero**: Antes de iniciar novo pagamento, o sistema verifica e resolve qualquer transação anterior pendente  
**Para que**: O terminal não fica bloqueado por transação anterior inacabada

**Cenário**: Terminal bloqueado por transação anterior
- **Dado**: Terminal retorna erro "1001 Waiting confirm" (Monri)
- **Quando**: Caixa tenta iniciar novo pagamento
- **Então**: Sistema detecta a transação anterior pendente
- **E**: Sistema recupera seu resultado do fornecedor
- **E**: Sistema confirma ou rejeita a transação anterior
- **E**: Apenas após resolução o novo pagamento é aceito

**Critérios de Aceitação**:
- Erro 1001 é classificado como "recovery signal", não como erro de novo pagamento
- Sistema bloqueia novo pagamento enquanto transação anterior está em recovery
- Transação anterior é identificada por paymentLineId + terminalId
- Resultado anterior é recuperado sem nova venda ao fornecedor
- Mensagem ao caixa: "Resultado de pagamento anterior está sendo verificado. Aguarde."

---

#### EFT-1003: Confirmar vs. Rejeitar Transação Recuperada

**Como**: EFT Device Service  
**Quero**: Decidir deterministicamente se confirmo ou rejeito uma transação recuperada  
**Para que**: Não há ambiguidade sobre qual foi o resultado final

**Cenário**: Transação aprovada, confirmação perdida
- **Dado**: Monri retorna "Approved" com approval code e amount
- **E**: A resposta é lida e validada
- **E**: Valor bate com tentativa original
- **Quando**: Serviço envia Confirm ao Monri
- **Então**: Monri libera o terminal
- **E**: Transação é marcada como "Confirmed"
- **E**: Resultado é publicado para MPOS Air como "OnSuccess"

**Cenário**: Transação com dados corrompidos
- **Dado**: Resposta de transação está incompleta/corrompida
- **Quando**: Serviço valida a resposta
- **Então**: Serviço não consegue determinar resultado com segurança
- **E**: Transação é marcada como "Recovery/Escalation"
- **E**: Nunca é enviado Reject (risco de reverter transação certa)
- **E**: Operação escalada para suporte

**Critérios de Aceitação**:
- Resposta é validada antes de Confirm/Reject
- Valor recuperado deve corresponder ao valor original (tolerance: 0)
- Approval code ou decline reason deve estar presente
- Confirm é enviado apenas para respostas válidas e retentáveis
- Reject é apenas para casos onde transação inacabada é identificada mas não pode ser aceita
- Log contém timestamp de Confirm/Reject e reconhecimento do fornecedor

---

#### EFT-1004: Recuperação Após Restart do Serviço

**Como**: EFT Device Service  
**Quero**: Na inicialização, procurar por transações em-voo que não foram resolvidas  
**Para que**: Não deixo pagamentos órfãos

**Cenário**: Serviço reinicia durante pagamento
- **Dado**: Serviço falha durante processamento de transação
- **E**: Transação foi persistida, mas Confirm/Reject não foi enviado
- **Quando**: Serviço reinicia
- **Então**: Na inicialização, serviço consulta DB por transações com estado "EFTProcessing"
- **E**: Para cada uma, serviço recupera seu estado do Monri
- **E**: Se resultado pode ser determinado, serviço completa Confirm/Reject
- **E**: Se resultado é incerto, transação permanece em recovery até ser resolvida manualmente

**Critérios de Aceitação**:
- Startup recovery usa `paymentOrderId` e `terminalId` para identificar transações Monri
- Transações "EFTProcessing" são prioridade máxima no recovery
- Cada transação recuperada tem log de timestamp e ações tomadas
- Nenhuma transação é silenciosamente descartada

---

#### EFT-1005: Status Lookup Durante Recovery

**Como**: MPOS Air  
**Quero**: Quando chamada final não chegou, usar `GET /eft-payments/{id}` para verificar estado  
**Para que**: Posso decidir se resultado é desconhecido ou final

**Cenário**: MPOS Air perde evento final, faz lookup
- **Dado**: `GET /eft-payments/{paymentId}` é chamado
- **Quando**: Transação está em estado "Recovery/OutcomeUnknown"
- **Então**: Resposta inclui
  - state: "OutcomeUnknown"
  - message: "Payment outcome is being checked. Do not retry."
  - lastStatusTime: timestamp da última atualização
- **E**: MPOS Air não cria novo pagamento, apenas sonda até status final

**Cenário**: Recovery foi completada
- **Dado**: Mesmo `GET /eft-payments/{paymentId}` após recovery completa
- **Quando**: Transação foi confirmada
- **Então**: Resposta inclui
  - state: "Completed"
  - outcome: "Approved" ou "Declined"
  - approval_code: código de autorização (se aprovado)
  - amount: valor confirmado

**Critérios de Aceitação**:
- Status nunca bloqueia em operação de longa duração
- Estado vem de persistência do serviço, não de poll do fornecedor
- MPOS Air pode pedir status, receber "OutcomeUnknown", esperar, pedir novamente

---

### Métricas de Sucesso do Épico

- Taxa de pagamentos recuperados com sucesso: >99%
- Tempo médio de recuperação: <2 minutos
- Zero transações órfãs em 8 semanas
- Zero duplicatas criadas durante recovery
- Taxa de sucesso de Confirm/Reject: 100%

---

## ÉPICO 2: IDENTIDADE E IDEMPOTÊNCIA

**ID**: EFT-EP-002  
**Prioridade**: Critical  
**Esforço estimado**: 5 pontos  
**Owner**: EFT Squad  

### Objetivo do Épico

Garantir que cada transação tem identidade única e estável. Um retry de transporte retorna o estado anterior sem enviar nova venda ao fornecedor.

### User Stories

#### EFT-2001: Gerar e Retornar externalId

**Como**: EFT Device Service  
**Quero**: Gerar um externalId único por tentativa e retornar no response de criação  
**Para que**: Cliente pode reenviar requisição com mesmo externalId e recebe o estado anterior

**Cenário**: Retry de criação de pagamento
- **Dado**: `POST /eft-payments` é enviado com paymentOrderId, amount
- **E**: Serviço cria tentativa com externalId = UUID gerado
- **E**: externalId é persistido em DB
- **Quando**: Response retorna com status 201
- **Então**: Response inclui `{ paymentId, externalId, status: "Initiated" }`
- **E**: Cliente recebe externalId

**Cenário**: Retry da mesma requisição
- **Dado**: Cliente reenvia `POST /eft-payments` com mesmo externalId no header `X-Idempotency-Key`
- **Quando**: Servidor recebe a requisição
- **Então**: Servidor não cria nova tentativa
- **E**: Servidor retorna a tentativa existente com seu estado atual
- **E**: Nenhuma chamada nova é feita ao Monri

**Critérios de Aceitação**:
- externalId é UUID (único, não predizível)
- externalId é retornado em cada resposta de criação
- externalId pode ser passado em header `X-Idempotency-Key` em retry
- Servidor verifica `X-Idempotency-Key` antes de criar nova tentativa
- Se header está presente e corresponde, retorna tentativa existente

---

#### EFT-2002: Diferenciar paymentLineId de externalId

**Como**: Arquiteto EFT  
**Quero**: Deixar claro que paymentLineId (negócio) é diferente de externalId (técnico)  
**Para que**: Múltiplas tentativas de mesmo pagamento têm mesmo paymentLineId, mas externalIds diferentes

**Cenário**: Cliente tenta pagar após recusa
- **Dado**: Primeira tentativa tem paymentLineId = 12345, externalId = UUID-A
- **E**: Monri retorna "Declined"
- **Quando**: Caixa tenta novamente com mesmo paymentLineId = 12345
- **Então**: Nova tentativa tem paymentLineId = 12345, externalId = UUID-B
- **E**: Ambas tentativas aparecem no histórico da linha de pagamento

**Critérios de Aceitação**:
- paymentLineId é de negócio, múltiplos EFT payments podem ter o mesmo
- externalId é único por tentativa, nunca reutilizado
- Documentação deixa claro essa diferença
- API always returns both in responses

---

#### EFT-2003: Rejeitar Duplicata Durante Processamento

**Como**: EFT Device Service  
**Quero**: Se tentativa está em processamento e mesmo externalId chegar novamente, retornar estado atual sem ação  
**Para que**: Não há race condition

**Cenário**: Duas requisições concorrentes
- **Dado**: Primeira requisição começa processamento com externalId = UUID-A
- **Quando**: Segunda requisição chega simultaneamente com mesmo externalId
- **Então**: Serviço detecta concorrência
- **E**: Segunda requisição retorna o estado da primeira (que está "EFTProcessing")
- **E**: Nenhuma chamada duplicada ao Monri

**Critérios de Aceitação**:
- Deduplicação por externalId usa lock pessimista ou CAS atomicamente
- Resposta inclui estado atual: "EFTProcessing", "Completed", etc
- Cliente pode pedir de novo sem impacto

---

### Métricas de Sucesso do Épico

- Taxa de duplicatas de transação criadas: 0%
- Taxa de sucesso de retry idempotente: 100%
- Latência média de retry: <500ms

---

## ÉPICO 3: PRONTIDÃO DO TERMINAL

**ID**: EFT-EP-003  
**Prioridade**: Critical  
**Esforço estimado**: 5 pontos  
**Owner**: EFT Squad  

### Objetivo do Épico

Fornecer status claro do terminal. Caixa pode verificar antes de iniciar venda. Diferentes estados (Ready, Busy, NotReady) têm significados claros.

### User Stories

#### EFT-3001: Endpoint GET /device-connection-status

**Como**: MPOS Air  
**Quero**: Chamar GET /device-connection-status para verificar se posso iniciar venda  
**Para que**: Sou bloqueado se terminal não está pronto

**Cenário**: Terminal pronto
- **Dado**: Terminal Monri está conectado, inicializado e idle
- **Quando**: MPOS Air chama `GET /device-connection-status`
- **Então**: Resposta retorna `{ status: "Connected", subStatus: "Ready", timestamp: "2026-09-29T14:30:00Z" }`

**Cenário**: Terminal ocupado
- **Dado**: Terminal está processando outra transação ou fechando
- **Quando**: MPOS Air chama `GET /device-connection-status`
- **Então**: Resposta retorna `{ status: "Connected", subStatus: "Busy", waitEstimate: 30, message: "Terminal processing another transaction" }`

**Cenário**: Terminal desconectado
- **Dado**: Terminal perdeu conexão com POS
- **Quando**: MPOS Air chama `GET /device-connection-status`
- **Então**: Resposta retorna `{ status: "Disconnected", message: "Terminal not reachable" }`

**Critérios de Aceitação**:
- Endpoint existe e é rápido (<500ms)
- Retorna sempre estado atual, nunca bloqueia
- Estados incluem: Connected + Ready/Busy/NeedInitialization, Disconnected, Initializing
- Substatuses são claros para UI/caixa

---

#### EFT-3002: Bloquear Venda Se Terminal Não Está Ready

**Como**: EFT Device Service  
**Quero**: Rejeitar `POST /eft-payments` se terminal não está em estado Ready  
**Para que**: Caixa não inicia venda destinada a falhar

**Cenário**: Terminal está pronto
- **Dado**: Terminal está Connected + Ready
- **Quando**: MPOS Air envia `POST /eft-payments`
- **Então**: Serviço aceita e processa

**Cenário**: Terminal está ocupado
- **Dado**: Terminal está Connected + Busy
- **Quando**: MPOS Air envia `POST /eft-payments`
- **Então**: Serviço retorna 429 ou 503
- **E**: Response inclui `{ error: "TERMINAL_BUSY", retry_after: 30, message: "Terminal processing another operation" }`

**Critério de Aceitação**:
- Validação de readiness acontece imediatamente após recebimento de requisição
- Não há race condition entre status check e venda
- Mensagem de erro é clara e acionável

---

#### EFT-3003: Status Sob Transição (Initializing)

**Como**: MPOS Air  
**Quero**: Saber quando terminal está inicializando (transição, não falha)  
**Para que**: Posso esperar em vez de exibir erro

**Cenário**: Terminal inicializando
- **Dado**: Terminal acabou de ser ligado
- **Quando**: MPOS Air chama `GET /device-connection-status`
- **Então**: Resposta retorna `{ status: "Initializing", progress: 50, message: "Terminal starting up" }`

**Critério de Aceitação**:
- Estado "Initializing" é claramente diferente de falha
- UI pode mostrar barra de progresso, não erro
- Timeout para inicialização é configurável (ex: 2 minutos)

---

### Métricas de Sucesso do Épico

- Taxa de verificações de status que evitam vendas condenadas: >90%
- Acurácia de status: 100%
- Latência de status: <500ms

---

## ÉPICO 4: RECEBIDA

**ID**: EFT-EP-004  
**Prioridade**: High  
**Esforço estimado**: 8 pontos  
**Owner**: EFT Squad + Country Team  

### Objetivo do Épico

Recebida padronizada com dados corretos, mascaramento de PAN, conformidade legal de cada país.

### User Stories

#### EFT-4001: Dados de Recebida do Fornecedor

**Como**: EFT Device Service  
**Quero**: Extrair dados de recebida da resposta de Monri  
**Para que**: Posso passar para MPOS Air decidir impressão

**Cenário**: Pagamento aprovado
- **Dado**: Monri retorna pagamento aprovado
- **Quando**: EFT Device Service processa resposta
- **Então**: Extrai e normaliza:
  - amount: valor em centavos
  - approval_code: código de autorização
  - card_masked: PAN mascarado (ex: 411111****1111)
  - timestamp: data/hora do terminal
  - transaction_reference: referência única Monri
  - terminal_id: identificador do terminal
  - merchant_name: nome do comerciante (se disponível)

**Critério de Aceitação**:
- Dados são extraídos da resposta Monri
- Nenhum dado sensível (PAN completo, PIN, track data) está incluído
- Todos os campos obrigatórios estão presentes
- Formato é consistente e reutilizável entre países

---

#### EFT-4002: Mascaramento de PAN

**Como**: EFT Device Service  
**Quero**: Mascarar PAN em todos os logs, respostas, recibos  
**Para que**: Dados de cartão nunca são expostos em texto limpo

**Cenário**: PAN mascarado
- **Dado**: Monri retorna PAN completo 4111111111111111
- **Quando**: EFT Device Service processa
- **Então**: Retorna 411111****1111
- **E**: Log contém apenas versão mascarada
- **E**: Resposta API contém apenas versão mascarada

**Critérios de Aceitação**:
- Mascaramento é aplicado em todos os pontos: API, logs, DB, recibos
- Máscara mostra primeiros 6 e últimos 4 dígitos
- Nenhuma exceção para "debug" ou "support"
- PAN completo nunca deixa EFT Device Service

---

#### EFT-4003: Recebida Aprovada vs. Recusada

**Como**: MPOS Air  
**Quero**: Recibos diferentes para aprovação e recusa  
**Para que**: Cliente vê o resultado correto

**Cenário**: Pagamento aprovado
- **Dado**: Resultado é "Approved"
- **Quando**: MPOS Air constrói recebida
- **Então**: Inclui:
  - "PAGAMENTO APROVADO" (título)
  - Valor, referência autorização, timestamp
  - PAN mascarado, nome comerciante
  - "Obrigado por sua compra"

**Cenário**: Pagamento recusado
- **Dado**: Resultado é "Declined"
- **Quando**: MPOS Air constrói recebida
- **Então**: Inclui:
  - "PAGAMENTO RECUSADO" (título)
  - Motivo: "Cartão expirado" (por exemplo)
  - PAN mascarado
  - "Tente outro cartão ou outro meio de pagamento"

**Critério de Aceitação**:
- Diferentes estados têm diferentes layouts
- Mensagens são aprovadas por compliance de cada país

---

#### EFT-4004: Conformidade Legal por País

**Como**: BA Funcional  
**Quero**: Cada país define sua regra de cópia, campos obrigatórios, retenção  
**Para que**: Recibos estão legalmente conformes

**Cenário**: Conformidade Portugal
- **Dado**: Recebida deve incluir: 2 cópias (cliente + loja), NIF comerciante, montante, timestamp
- **Quando**: MPOS Air imprime
- **Então**: Ambas cópias têm todos os dados obrigatórios

**Cenário**: Conformidade Hungria
- **Dado**: Recebida pode ter requisitos diferentes (ex: retenção 5 anos)
- **Quando**: Suporte resgata recebida antiga
- **Então**: Sistema tem todas as cópias históricas

**Critério de Aceitação**:
- Cada país tem definição escrita de conformidade
- MPOS Air implementa conforme à regra local
- Não há assunção de outro país

---

### Métricas de Sucesso do Épico

- Taxa de conformidade de recibas: 100%
- Taxa de recibas com PAN mascarado: 100%
- Zero exposições de PAN em logs

---

## ÉPICO 5: MENSAGENS DE PROGRESSO

**ID**: EFT-EP-005  
**Prioridade**: High  
**Esforço estimado**: 3 pontos  
**Owner**: EFT Squad + MPOS Air  

### Objetivo do Épico

EFT Device Service emite chaves de evento padronizadas. MPOS Air decide o texto localizado aprovado.

### User Stories

#### EFT-5001: Eventos de Progresso Padronizados

**Como**: EFT Device Service  
**Quero**: Emitir eventos padronizados durante pagamento (PRESENT_CARD, PIN_REQUIRED, AUTHORIZING)  
**Para que**: Caixa sempre sabe o que terminal está fazendo

**Cenário**: Fluxo de pagamento normal
- **Dado**: Pagamento foi iniciado no Monri
- **Quando**: Terminal aguarda inserção de cartão
- **Então**: EFT emite evento `{ event: "PRESENT_CARD", timestamp: "..." }`
- **E**: MPOS Air mostra "Insira cartão"

**Quando**: Terminal aguarda PIN
- **Então**: EFT emite evento `{ event: "PIN_REQUIRED" }`
- **E**: MPOS Air mostra "Digite PIN no terminal"

**Quando**: Terminal está autorizando
- **Então**: EFT emite evento `{ event: "AUTHORIZING" }`
- **E**: MPOS Air mostra "Autorizando pagamento. Por favor aguarde."

**Critérios de Aceitação**:
- Eventos são emitidos via SignalR ou callback configurado
- Evento contém timestamp e identificador de tentativa
- Eventos são baseados em progresso de Monri, mapeados para chaves genéricas
- Não há valor, autorização ou dados sensíveis nos eventos

---

#### EFT-5002: Event de Recovery

**Como**: EFT Device Service  
**Quero**: Emitir evento RECOVERY_REQUIRED quando resultado é desconhecido  
**Para que**: Caixa sabe que não deve tentar novamente

**Cenário**: Comunicação caiu
- **Dado**: Response perdida após despacho
- **Quando**: EFT detecta a situação
- **Então**: Emite evento `{ event: "RECOVERY_REQUIRED", message: "Payment outcome is being checked. Do not retry." }`
- **E**: MPOS Air mostra mensagem bloqueadora
- **E**: Caixa não consegue iniciar novo pagamento

**Critério de Aceitação**:
- Mensagem é clara: "NÃO TENTE NOVAMENTE"
- UI bloqueia ação, nunca apenas aviso
- Evento é emitido em tempo real

---

### Métricas de Sucesso do Épico

- Taxa de compreensão de mensagens: >95% (usuários entendem o que terminal está fazendo)
- Taxa de "contato suporte" por confusão de mensagem: <2%

---

## ÉPICO 6: MAPEAMENTO DE ERROS

**ID**: EFT-EP-006  
**Prioridade**: High  
**Esforço estimado**: 5 pontos  
**Owner**: EFT Squad  

### Objetivo do Épico

Mapear códigos de erro de cada fornecedor para chaves normalizadas. Única coluna que muda é o código nativo.

### User Stories

#### EFT-6001: Mapeamento Monri Completo

**Como**: BA Funcional  
**Quero**: Ter tabela de mapeamento que traduz 150+ códigos Monri para 7 chaves normalizadas  
**Para que**: Cada erro Monri tem comportamento claro em MPOS Air

**Mapeamento exemplo:**

| Código Monri | Categoria Arquitetura | Chave Normalizada | Mensagem MPOS Air |
|---|---|---|---|
| 1 | Device Error | TERMINAL_NOT_READY | O terminal de pagamento não está pronto. Use outro método ou contato suporte. |
| 2 | Busy | TERMINAL_BUSY | O terminal está processando outra operação. Aguarde. |
| 4 | Busy | TERMINAL_BUSY | O terminal está processando outra operação. Aguarde. |
| 1001 | Recovery Signal | PAYMENT_OUTCOME_CHECKING | O resultado do pagamento está sendo verificado. Não tente novamente. |
| 1014 | Declined | PAYMENT_DECLINED | Pagamento não autorizado. Use outro cartão ou método. |
| 1030 | User Cancel | PAYMENT_CANCELLED | Pagamento foi cancelado. |
| 11 | User Cancel | PAYMENT_CANCELLED | Pagamento foi cancelado. |

**Critérios de Aceitação**:
- Todos os 150+ códigos Monri são mapeados
- Zero código sem mapeamento (unmapped → recovery escalation)
- Tabela é mantida como documento vivo em Confluence
- Cada entrada tem categoria, chave e mensagem

---

#### EFT-6002: Framework de Mapeamento para Novo Fornecedor

**Como**: BA Funcional + Integration Architect  
**Quero**: Quando novo fornecedor entra, apenas mapear suas códigos para as mesmas 7 chaves  
**Para que**: Não preciso redesenhar framework inteiro

**Cenário**: Novo fornecedor entra
- **Dado**: Fornecedor X usa códigos próprios (ex: E001, E002, E003)
- **Quando**: Documentação de fornecedor X é recebida
- **Então**: Mapeio apenas códigos de X para as 7 chaves:
  - E001 (Device Error) → TERMINAL_NOT_READY
  - E002 (Busy) → TERMINAL_BUSY
  - E003 (Declined) → PAYMENT_DECLINED
- **E**: Resto do código permanece igual

**Critério de Aceitação**:
- Novo mapeamento leva <4h (não <1 dia)
- Zero mudança no código MPOS Air
- Framework permanece estável

---

#### EFT-6003: Erro Desconhecido Falha Seguramente

**Como**: EFT Device Service  
**Quero**: Se recebo código que nunca foi visto, não adivinhar; falhar em segurança com recovery  
**Para que**: Nenhuma entrada desconhecida passa silenciosamente

**Cenário**: Código novo/não mapeado
- **Dado**: Monri retorna código 9999 (nunca antes visto)
- **Quando**: Serviço recebe e tenta mapear
- **Então**: Não encontra mapeamento
- **E**: Transação vai para recovery/escalação
- **E**: Log contém código original 9999 como diagnóstico
- **E**: Não tenta nenhuma adivinhação
- **E**: Support investiga e cria novo mapeamento

**Critério de Aceitação**:
- Código desconhecido nunca retorna ao POS sem mapping
- Log contém código original para análise
- Zero "sorte" em assumir significado

---

### Métricas de Sucesso do Épico

- Taxa de cobertura de mapeamento: 100% (todos os códigos conhecidos mapeados)
- Tempo para novo fornecedor entrar: <1 dia
- Taxa de escalação por "código desconhecido": <0.1%

---

## ÉPICO 7: OBSERVABILIDADE E SUPORTE

**ID**: EFT-EP-007  
**Prioridade**: High  
**Esforço estimado**: 8 pontos  
**Owner**: EFT Squad + DevOps  

### Objetivo do Épico

Support consegue procurar qualquer transação e ver sequência completa de eventos, estados, tentativas.

### User Stories

#### EFT-7001: Searchable Logs por Payment ID

**Como**: Support  
**Quero**: Procurar uma transação por `paymentId` ou `externalId` e ver sequência completa  
**Para que**: Posso diagnosticar sem acessar código

**Cenário**: Support procura transação
- **Dado**: Cliente relata "Pagamento desapareceu"
- **Quando**: Support procura por `paymentId=abc123` em dashboard
- **Então**: Dashboard mostra:
  - Timestamp de criação
  - Sequência de estados: Initiated → EFTProcessing → OutcomeUnknown → Confirmed → Completed
  - Cada timestamp
  - Tentativas de Confirm/Reject
  - Resultado final
  - PAN mascarado (para contexto)
  - Terminal ID

**Critério de Aceitação**:
- Dashboard permite search por paymentId, externalId, terminalId
- Timeline é legível com tempos exatos
- Nenhum dado sensível está exposto (PAN, PIN, track)

---

#### EFT-7002: Runbook Terminal Bloqueado

**Como**: Support  
**Quero**: Ter guia claro de ações quando terminal está bloqueado (erro 1001)  
**Para que**: Resisto 80% dos casos sem escalar

**Runbook exemplo:**
```
Terminal está bloqueado (erro 1001 - Waiting confirm)

1. Procurar transação em recovery: SELECT * FROM eft_payments WHERE terminal_id = X AND state = 'Recovery'
2. Verificar se resultado pode ser determinado (resposta Monri é completa)
3. Se sim: Enviar Confirm (assume aprovação se dados batem)
4. Se não: Enviar Reject (reverter para ser seguro)
5. Verificar terminal status após ação
6. Se ainda bloqueado: Escalar a Integration Architect + Monri
```

**Critério de Aceitação**:
- Runbook é documentado e mantido atualizado
- Support consegue seguir sem código

---

#### EFT-7003: Observabilidade de Metrics

**Como**: DevOps  
**Quero**: Métricas de taxa de sucesso, tempo médio, taxa de recovery  
**Para que**: Posso detectar degradação antes que cliente reclame

**Métricas**:
- Payment success rate (% approved)
- Payment decline rate (% declined)
- Recovery rate (% outcome unknown)
- Average payment time (ms)
- Average recovery time (s)
- P99 latency
- Error rate (% failed due to service issue)
- Terminal ready rate (%)

**Critério de Aceitação**:
- Métricas são emitidas para observabilidade (ex: Prometheus)
- Alertas estão configurados para anomalia
- Dashboard mostra trending

---

### Métricas de Sucesso do Épico

- Taxa de resolução sem escalação: >80%
- Tempo médio de resolução com observabilidade: <30 min
- Satisfação de support: >8/10

---

## ÉPICO 8: CONTRATO MPOS AIR ESTÁVEL

**ID**: EFT-EP-008  
**Prioridade**: Critical  
**Esforço estimado**: 2 pontos  
**Owner**: EFT Squad + MPOS Air Lead  

### Objetivo do Épico

Contrato entre MPOS Air e EFT Device Service é tratado como fixo. Mudanças são raras e explícitas.

### User Stories

#### EFT-8001: Contrato Core é Versionado

**Como**: Arquiteto EFT  
**Quero**: Versionar contrato core (POST /eft-payments, GET /eft-payments/{id}, GET /device-connection-status)  
**Para que**: Alterações futuras são explícitas e negociadas

**Contrato v1 (Baseline):**
```
POST /eft-payments
Request: paymentOrderId, paymentLineId, storeNumber, tillNumber, userId, amount
Response: paymentId, externalId, status

GET /eft-payments/{id}
Response: id, state, error.code, error.additionalInformation

GET /device-connection-status
Response: status, subStatus, message
```

**Critério de Aceitação**:
- Contrato é documentado e versionado
- Mudanças futuras requerem aprovação explicita
- Breaking changes vão para v2 (não mudam v1)
- Documentação deixa claro qual versão está em produção

---

#### EFT-8002: Estabilidade em 8 Semanas

**Como**: Product Owner  
**Quero**: Que contrato core não tenha breaking changes nos primeiros 8 semanas de produção  
**Para que**: Confiança em arquitetura

**Métrica**: Zero breaking changes ao contrato documentado.

**Critério de Aceitação**:
- Quaisquer mudanças planejadas são comunicadas 2 semanas antes
- Mudanças não breaking são backward compatible
- Breaking changes (se houver) vão para versão nova

---

### Métricas de Sucesso do Épico

- Taxa de breaking changes: 0% nos primeiros 8 semanas
- Tempo de comunicação de mudança: 2 semanas de aviso prévio

---

## ROADMAP DE INTEGRAÇÃO POR ÉPICO

### Ordem Recomendada de Implementação

| Fase | Épicos | Duração | Dependências |
|------|--------|---------|--------------|
| **Fase 1: Core** | EP-008 (Contrato), EP-002 (Idempotência), EP-003 (Prontidão) | 2 semanas | Nenhuma |
| **Fase 2: Payment Flow** | EP-001 (Recovery), EP-006 (Error Mapping) | 3 semanas | Fase 1 |
| **Fase 3: UX** | EP-005 (Mensagens), EP-004 (Recebida) | 2 semanas | Fase 2 |
| **Fase 4: Operações** | EP-007 (Observabilidade) | 2 semanas | Todas |

**Total estimado**: 9 semanas para todos os épicos.

---

## PRÓXIMOS PASSOS

1. **Preparar para Jira**: Cada épico vira epic story no projeto MAIR
2. **Refinar stories**: Adicionar estimate em story points, assignees
3. **Priorizar por país**: Monri/Croatia é P1, Hungary/Romania aguardam fornecedor
4. **Criar dependências**: Mapear quem bloqueia quem
5. **Atribuir owner**: Cada épico tem tech lead + BA responsável

---

**Versão**: 1.0  
**Data**: 29 de setembro de 2026  
**Status**: DRAFT - Aguardando aprovação de Ana Maria Gropeneanu e Markéta Janáková
