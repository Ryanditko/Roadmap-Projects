# Serviço Mock de Processamento de Pagamentos

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um serviço que se comporta como um gateway de pagamentos — Stripe ou Adyen — sem nunca tocar em uma rede de cartões real. Clientes enviam uma cobrança, seu serviço a valida, a move por um ciclo de vida de transação (autorizada → capturada → liquidada, ou recusada) e depois dispara um webhook informando ao lojista o que aconteceu. O dinheiro é falso, mas os problemas difíceis são reais: fazer com que uma requisição repetida cobre o cliente exatamente uma vez, modelar uma máquina de estados que nunca pode pular uma etapa e entregar webhooks de forma confiável a um receptor que pode estar fora do ar. É aqui que você aprende por que chaves de idempotência existem e por que código de pagamento é escrito de forma tão defensiva.

## Pré-requisitos

- Conforto para construir e versionar endpoints REST ([API de E-commerce com JWT](../01-ecommerce-api-jwt/) combina bem como consumidora deste serviço)
- Um banco de dados relacional e entendimento de transações e restrições de unicidade
- Familiaridade com a semântica dos status HTTP e cabeçalhos de requisição/resposta
- Noção básica de máquinas de estados finitos e por que transições ilegais importam

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar chaves de idempotência para que uma requisição duplicada nunca cobre duas vezes
- Modelar um pagamento como uma máquina de estados explícita com transições guardadas
- Projetar lógica de estorno que não pode exceder o valor capturado
- Entregar webhooks de forma confiável com assinaturas, retentativas e semântica at-least-once
- Explicar, em nível conceitual, por que sistemas reais nunca armazenam dados brutos de cartão (escopo PCI-DSS)

## Requisitos Funcionais

1. O sistema deve aceitar uma requisição de cobrança com um `Idempotency-Key`; repetir a mesma chave deve retornar o resultado original, não criar uma nova cobrança.
2. Uma cobrança deve ser validada (valor > 0, moeda suportada, token de cartão bem-formado) e rejeitada com 4xx antes que qualquer estado seja criado.
3. Cada pagamento deve progredir por um ciclo de vida definido; o sistema deve rejeitar qualquer transição que não seja legal a partir do estado atual.
4. O sistema deve simular recusas de forma determinística (ex.: um token de cartão de teste mágico sempre falha) para que clientes testem caminhos de falha.
5. Um estorno nunca deve exceder o saldo estornável restante, e estornos parciais devem acumular corretamente.
6. A cada mudança de estado o sistema deve enfileirar um webhook para a URL registrada do lojista, assinado para que o receptor verifique a autenticidade.
7. Entregas de webhook que falharem devem ser retentadas com backoff exponencial até um número limitado de tentativas.
8. Cada mudança de estado deve ser registrada em um log de auditoria somente-adição, que nunca é modificado.

## Marcos Sugeridos

1. **Marco 1 — Cobrança e idempotência:** Aceite uma cobrança, persista-a e faça o `Idempotency-Key` retornar o resultado armazenado na repetição.
2. **Marco 2 — Ciclo de vida e estornos:** Implemente a máquina de estados, transições de captura/liquidação, recusas determinísticas e estornos limitados.
3. **Marco 3 — Webhooks e auditoria:** Assine e entregue webhooks com retentativa/backoff e escreva uma trilha de auditoria imutável para cada transição.

## Esboço de Dados e Interface

```text
Payment
  id:               string
  idempotencyKey:   string (único)
  amount:           inteiro (unidades menores, ex.: centavos)
  currency:         string (ISO 4217, ex.: "USD")
  status:           enum { requires_capture, captured, settled, declined, refunded, partially_refunded }
  refundedAmount:   inteiro
  createdAt:        string ISO-8601

Transições legais:
  requires_capture -> captured -> settled
  requires_capture -> declined
  captured|settled -> partially_refunded -> refunded

POST /payments        headers: Idempotency-Key: <uuid>
                      body: { amount, currency, cardToken }
                      -> 201 { id, status } | 200 (repetido) | 402 recusado
POST /payments/{id}/refund   body: { amount }
                      -> 200 { status, refundedAmount } | 422 estorno excessivo
GET  /payments/{id}   -> 200 { ...payment } | 404

Webhook -> URL do lojista
  headers: X-Signature: hmac-sha256(secret, body)
  body:    { event: "payment.captured", paymentId, status }
```

## Desafios Extras

- Adicione uma simulação de 3D Secure: alguns tokens exigem uma etapa extra de confirmação antes da captura.
- Implemente um relatório de reconciliação que some capturas menos estornos por dia.
- Suporte múltiplas moedas com uma tabela de conversão fixa e armazene a taxa usada.
- Adicione um endpoint de reenvio de webhook para que lojistas possam re-solicitar eventos passados por ID.

## Definição de Pronto

- [ ] Repetir uma requisição com a mesma chave de idempotência retorna a resposta idêntica e não cria uma segunda cobrança.
- [ ] Toda transição de estado ilegal é rejeitada; apenas caminhos definidos têm sucesso.
- [ ] Estornos são limitados ao saldo estornável e estornos parciais somam exatamente.
- [ ] Webhooks são assinados, retentados com backoff e desistem após um número limitado de tentativas.
- [ ] O log de auditoria é somente-adição e reflete cada transição em ordem.

## Armadilhas Comuns

- Tratar a chave de idempotência como opcional — verifique-a dentro da mesma transação que insere a cobrança, ou uma condição de corrida cria duas.
- Armazenar números de cartão "só para o mock". Guarde um token opaco; manter PANs reais te coloca no escopo PCI sem motivo.
- Tornar as recusas aleatórias em vez de determinísticas, o que deixa os testes dos clientes instáveis.
- Disparar webhooks de forma síncrona dentro da requisição de cobrança, fazendo um lojista lento bloquear pagamentos — enfileire e entregue fora do fluxo.
- Usar floats para dinheiro. Unidades inteiras menores evitam desvios de arredondamento.

## Recursos

- [Stripe: Requisições idempotentes](https://docs.stripe.com/api/idempotent_requests) — o modelo canônico para retentativas seguras.
- [Stripe: Webhooks](https://docs.stripe.com/webhooks) — assinatura, retentativas e design de eventos.
- [PCI Security Standards Council](https://www.pcisecuritystandards.org/) — o que o tratamento de dados de cartão realmente exige.
- [Martin Fowler: State Machine](https://martinfowler.com/dslCatalog/stateMachine.html) — modelando transições guardadas.
- [RFC 4122: UUIDs](https://datatracker.ietf.org/doc/html/rfc4122) — gerando chaves de idempotência sem colisão.
