# Sistema de Transações Financeiras Idempotente

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Em finanças, "a requisição expirou" é a frase mais assustadora que existe: a transferência aconteceu ou não? O cliente vai repetir, e se o seu sistema processar esse retry como uma segunda transferência, você acabou de mover o dinheiro de alguém duas vezes. Este projeto pede que você construa um sistema de transações onde a semântica **exactly-once** se mantém mesmo que a rede garanta apenas at-least-once. O mecanismo é a chave de idempotência: o cliente carimba cada operação lógica com uma chave única, e o servidor promete que reprocessar a mesma chave retorna o resultado original sem repetir o efeito colateral. Em torno disso você construirá uma máquina de estados de transação, contabilidade de partidas dobradas que nunca deixa dinheiro aparecer ou sumir, estornos para os casos que precisam ser desfeitos e uma trilha de auditoria em que um contador confiaria.

## Pré-requisitos

- Experiência sólida construindo APIs REST transacionais apoiadas por um banco relacional
- Domínio firme de transações de banco, níveis de isolamento e restrições de unicidade
- Entendimento de perigos de concorrência (condições de corrida, lost updates, double-submit)
- Familiaridade com comportamento de retry em clientes e filas de mensagens
- Uma stack de backend de sua escolha além de um datastore com transações reais (Postgres, MySQL, etc.)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um protocolo de chave de idempotência que torna retries seguros de ponta a ponta
- Modelar uma transação como uma máquina de estados com transições legais e auditáveis
- Implementar contabilidade de partidas dobradas para que débitos e créditos sempre fechem
- Lidar com requisições duplicadas concorrentes de forma determinística sob níveis de isolamento reais
- Implementar estornos e reconciliação sem quebrar os invariantes do razão (ledger)
- Construir uma trilha de auditoria imutável capaz de reconstruir qualquer saldo em qualquer ponto no tempo

## Requisitos Funcionais

1. Toda operação que muta estado deve aceitar uma chave de idempotência fornecida pelo cliente; reprocessar a mesma chave deve retornar o resultado original e nunca repetir o efeito colateral.
2. Uma transação deve percorrer uma máquina de estados explícita (ex.: `pendente → processando → liquidada | falha | estornada`) permitindo apenas transições legais.
3. O razão deve usar contabilidade de partidas dobradas: toda transação lança entradas de débito e crédito balanceadas, e os saldos totais devem sempre reconciliar a zero.
4. Requisições concorrentes com a mesma chave de idempotência devem ser serializadas para que exatamente uma execute o efeito (imposto por uma restrição de unicidade ou lock, não apenas lógica de aplicação).
5. O sistema deve suportar estornos que lançam entradas compensatórias em vez de deletar ou mutar registros originais.
6. Um log de auditoria imutável e append-only deve registrar toda transição de estado e lançamento com ator, timestamp e motivo.
7. Uma rotina de reconciliação deve verificar que os saldos de conta equivalem à soma de suas entradas no razão e sinalizar qualquer divergência.
8. **Consistência:** dinheiro nunca deve ser criado nem destruído; toda operação é atômica e o invariante do razão se mantém após qualquer crash ou retry.
9. **Confiabilidade:** uma operação interrompida no meio (crash entre cobrar e liquidar) deve ser recuperável a um estado terminal correto, não deixada ambígua.

## Marcos Sugeridos

1. **Marco 1 — Razão e partidas dobradas:** Modele contas e entradas balanceadas; imponha que toda transação feche atomicamente.
2. **Marco 2 — Chaves de idempotência:** Adicione o protocolo de chave; armazene chave → resultado e faça replays retornarem o desfecho armazenado.
3. **Marco 3 — Máquina de estados e concorrência:** Implemente transições legais e prove que requisições concorrentes de mesma chave produzem exatamente um efeito.
4. **Marco 4 — Estornos e auditoria:** Adicione estornos compensatórios e uma trilha de auditoria append-only.
5. **Marco 5 — Reconciliação e recuperação:** Reconcilie saldos contra entradas e recupere transações interrompidas a um estado terminal.

## Esboço de Dados e Interface

```text
Conta               Entrada do Razão (partida dobrada)
  id, balance         id, txId, accountId, direction(debito|credito), amount

Transação
  id, idempotencyKey (UNIQUE), state, amount, from, to, createdAt

Registro de idempotência
  key (UNIQUE), requestHash, responseSnapshot, status(em_progresso|feito)

Máquina de estados
  pendente -> processando -> liquidada
                          \-> falha
  liquidada -> estornada   (lança entradas compensatórias; original intacto)

POST /transfers
  Header: Idempotency-Key: <uuid>
  body { from, to, amount }
  -> 201 { txId, state }        (primeira vez)
  -> 200 { txId, state }        (replay: mesmo resultado, sem novo efeito)
  -> 409                        (mesma chave, corpo de requisição diferente)

Invariante verificado em todo commit:  Σ débitos == Σ créditos
```

## Desafios Extras

- Adicione uma janela de liquidação que agrupa transações pendentes e as liquida juntas.
- Suporte contas multi-moeda com entradas de câmbio explícitas que ainda balanceiam.
- Adicione checagens básicas de fraude/velocidade que possam segurar uma transação em `pendente` para revisão.
- Exponha uma consulta de saldo em um ponto no tempo reprocessando o razão até um timestamp.

## Definição de Pronto

- [ ] Reprocessar qualquer requisição com a mesma chave de idempotência retorna o resultado original e não causa um segundo efeito.
- [ ] Requisições concorrentes de mesma chave resultam em exatamente uma transação lançada, imposto no datastore.
- [ ] Toda transação lança entradas balanceadas; o razão reconcilia a zero o tempo todo.
- [ ] Apenas transições de estado legais são permitidas; as ilegais são rejeitadas.
- [ ] Estornos lançam entradas compensatórias e nunca mutam ou deletam os originais.
- [ ] Um crash no meio de uma transação recupera a um estado terminal correto, sem dinheiro ambíguo ou duplicado.

## Armadilhas Comuns

- Impor idempotência apenas no código da aplicação — sem uma restrição de unicidade, duas requisições concorrentes passam ambas pela checagem e lançam em dobro.
- Tratar um timeout como falha e deixar o cliente repetir para uma duplicata, em vez de tornar o retry seguro.
- Mutar ou deletar entradas do razão para "corrigir" um erro em vez de lançar um estorno, destruindo a trilha de auditoria.
- Armazenar saldos como fonte da verdade em vez de derivá-los das entradas, de modo que a divergência se torna irrecuperável.
- Retornar uma resposta idempotente cacheada para um corpo de requisição *diferente* sob a mesma chave — detecte o descasamento e rejeite.
- Ignorar níveis de isolamento, de modo que um lost update deixa duas transferências lerem o mesmo saldo e sacar além do limite.

## Recursos

- [Stripe: Projetando APIs robustas e previsíveis com idempotência](https://stripe.com/blog/idempotency) — a referência sobre chaves de idempotência em uma API de pagamentos.
- [Martin Fowler: Accounting Patterns (Ledger)](https://martinfowler.com/eaaDev/AccountingNarrative.html) — modelagem de partidas dobradas para software.
- [PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html) — o que cada nível de isolamento de fato previne.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — capítulos sobre transações, isolamento e consistência.
- [Pat Helland: Life beyond Distributed Transactions](https://queue.acm.org/detail.cfm?id=3025012) — idempotência e at-least-once em escala.
