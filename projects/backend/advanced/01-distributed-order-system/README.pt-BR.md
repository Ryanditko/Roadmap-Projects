# Sistema Distribuído de Processamento de Pedidos

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Um pedido em um sistema de e-commerce real raramente toca um único banco de dados. Fazer um debita estoque, cobra um cartão e agenda uma entrega — três serviços separados, três armazenamentos separados, nenhuma transação compartilhada para uni-los. Este projeto pede que você torne esse fluxo *correto* mesmo assim: quando o pagamento tem sucesso mas a entrega não tem capacidade, o estoque reservado deve ser liberado e a cobrança estornada. Você construirá o fluxo como uma **saga** — uma sequência de transações locais em que cada passo tem uma ação compensatória que o desfaz. No caminho você encontra as verdades duras dos sistemas distribuídos: não há rollback global, a consistência é eventual, e toda chamada de rede pode falhar, expirar ou silenciosamente ter sucesso duas vezes.

## Pré-requisitos

- Experiência sólida com serviços REST/HTTP ([API Gateway](../../intermediate/09-api-gateway/) e [Fila de Tarefas com RabbitMQ](../../intermediate/04-job-queue-rabbitmq/) são bons aquecimentos)
- Conforto para rodar vários serviços localmente (Docker Compose ou equivalente)
- Familiaridade com um message broker ou fila durável
- Um modelo mental funcional de transações de banco (ACID) e por que elas não cruzam fronteiras de serviço
- Qualquer stack de backend que você goste (Node, Go, Java, Python, C#)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar um fluxo de negócio multisserviço como uma saga com compensações explícitas
- Escolher entre orquestração (um coordenador central) e coreografia (serviços reagindo a eventos) e justificar o trade-off
- Tornar cada passo idempotente para que retries e mensagens duplicadas sejam seguros
- Raciocinar sobre consistência eventual e expor estados intermediários honestamente aos clientes
- Rastrear um único pedido através das fronteiras de serviço e se recuperar de falhas parciais

## Requisitos Funcionais

1. O sistema deve aceitar um pedido e coordenar reserva de estoque, pagamento e entrega em pelo menos três serviços.
2. Cada passo deve ser uma transação local com uma ação compensatória definida (liberar estoque, estornar pagamento, cancelar entrega).
3. Uma falha em qualquer passo deve disparar compensações para todos os passos concluídos anteriormente, em ordem reversa.
4. Toda operação deve ser idempotente: reprocessar o mesmo comando ou evento não deve cobrar ou reservar em dobro.
5. O pedido deve expor uma máquina de estados observável (ex.: `pendente → confirmado → falho → compensando → cancelado`).
6. Mensagens falhas ou não processáveis devem cair em uma fila de mensagens mortas (dead-letter) em vez de serem perdidas ou repetidas para sempre.
7. **Confiabilidade:** o fluxo deve sobreviver a um crash de serviço no meio da saga e retomar ou compensar ao reiniciar — sem reservas órfãs.
8. **Consistência:** o design deve ser eventualmente consistente; documente quais janelas de inconsistência são aceitáveis e por quê.
9. **Observabilidade:** um único correlation ID deve permitir rastrear um pedido de ponta a ponta por todos os serviços.

## Marcos Sugeridos

1. **Marco 1 — O caminho feliz:** Três serviços, síncronos ou orientados a eventos, completando um pedido bem-sucedido. Ainda sem falhas.
2. **Marco 2 — Compensação:** Force uma falha em cada passo e implemente as transações compensatórias para que o sistema se autocorrija.
3. **Marco 3 — Idempotência e durabilidade:** Adicione chaves de idempotência, um log de saga durável e tratamento de dead-letter; mate um serviço no meio do fluxo e confirme que ele se recupera.
4. **Marco 4 — Observabilidade:** Adicione correlation IDs e tracing distribuído para acompanhar um pedido movendo-se pelo sistema.

## Esboço de Dados e Interface

```text
                 ┌─────────────┐
   POST /orders  │ Orquestrador│  persiste estado da saga + log de passos
 ───────────────▶│  de Saga    │
                 └──────┬──────┘
        reservar │ cobrar │ enviar     (cada passo: tentar + compensar)
        ┌───────▼──┐ ┌───▼────┐ ┌──────▼───────┐
        │ Estoque  │ │Pagamnto│ │   Entrega    │
        └──────────┘ └────────┘ └──────────────┘

Entrada de log de passo da saga
  orderId, stepName, status(pendente|feito|compensado), idempotencyKey

Estado do pedido: pendente -> confirmado
                          \-> falho -> compensando -> cancelado
```

## Desafios Extras

- Implemente o mesmo fluxo **duas vezes** — uma orquestrada, uma coreografada — e compare a facilidade de depuração.
- Adicione um timeout por passo para que um serviço travado dispare compensação em vez de pendurar para sempre.
- Introduza event sourcing para o agregado do pedido e reconstrua o estado a partir do log de eventos.
- Adicione uma pequena tela de admin listando sagas atualmente presas em `compensando`.

## Definição de Pronto

- [ ] Um pedido bem-sucedido transiciona limpamente por todos os passos até `confirmado`.
- [ ] Uma falha forçada em qualquer passo compensa todos os passos anteriores e termina em `cancelado`.
- [ ] Reprocessar qualquer comando ou evento não produz efeitos colaterais duplicados.
- [ ] Matar um serviço no meio da saga não deixa estoque permanentemente reservado nem cobrança sem estorno após a recuperação.
- [ ] Um único correlation ID rastreia um pedido em cada log de serviço.

## Armadilhas Comuns

- Tratar uma saga como uma transação de banco e esperar um rollback automático — não existe; você escreve cada compensação à mão.
- Esquecer que "a rede expirou" e "a operação falhou" são diferentes: a cobrança pode ter passado. Chaves de idempotência são o que te salvam.
- Compensar na ordem errada, ou compensar um passo que nunca de fato completou.
- Esconder o estado intermediário `pendente` dos clientes e fingir que o pedido é instantâneo.

## Recursos

- [Microservices.io: Padrão Saga](https://microservices.io/patterns/data/saga.html) — orquestração vs. coreografia, com exemplos trabalhados.
- [Microsoft: Padrão de transações distribuídas Saga](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga) — passo a passo em nível de arquitetura.
- [Hector Garcia-Molina & Kenneth Salem: "Sagas" (1987)](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) — o artigo original.
- [Stripe: Projetando APIs robustas e previsíveis com idempotência](https://stripe.com/blog/idempotency) — como chaves de idempotência funcionam na prática.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — capítulos sobre transações distribuídas e consistência.
