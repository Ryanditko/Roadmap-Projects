# Pipeline de CDC (Change Data Capture)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um pipeline de Change Data Capture que transmite cada insert, update e delete de um banco operacional para um store a jusante — mantendo uma réplica ou um data lake continuamente sincronizados sem martelar a origem com consultas de polling. A abordagem correta lê o write-ahead log do banco (CDC baseado em log via Debezium), que captura mudanças na ordem de commit e com baixo overhead. As partes genuinamente difíceis são a ordenação (mudanças na mesma linha devem aplicar em sequência), o handoff snapshot-depois-stream (carregar dados existentes e então trocar para o log sem lacuna nem duplicata), mudanças de schema no meio do stream, e idempotência para que um replay após um crash convirja ao mesmo estado. Você escolherá semânticas de entrega — at-least-once com upserts idempotentes costuma ser o alvo pragmático — e provará a convergência.

## Pré-requisitos

- Um banco com replicação lógica / acesso ao WAL (PostgreSQL, MySQL ou MongoDB)
- Familiaridade com Kafka ou outro log para transportar eventos de mudança
- Entendimento de chaves primárias, upserts e idempotência
- Conforto com os tradeoffs de entrega exactly-once vs at-least-once

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Explicar CDC baseado em log vs baseado em consulta vs baseado em trigger e escolher um
- Fazer um snapshot inicial consistente e passar para streaming sem lacunas ou duplicatas
- Preservar a ordenação por chave através do particionamento
- Tratar mudanças de schema na origem (colunas adicionadas/removidas/renomeadas) sem quebrar consumidores
- Projetar lógica de aplicação idempotente para que replays convirjam ao estado idêntico

## Requisitos Funcionais

1. O pipeline deve capturar inserts, updates e deletes da origem na ordem de commit.
2. Um snapshot inicial deve carregar as linhas existentes e então transicionar para streaming sem mudanças perdidas ou duplicadas na fronteira.
3. Mudanças na mesma chave primária devem ser aplicadas na sua ordem original a jusante.
4. Aplicar o stream de mudanças deve ser idempotente: reproduzir a partir de um offset anterior converge ao mesmo estado final.
5. Uma mudança de schema na origem deve ser detectada e propagada (ou rejeitada com segurança), não corromper silenciosamente o destino.
6. O lag de replicação (commit na origem → aplicado a jusante) deve ser exposto como métrica.

## Marcos Sugeridos

1. **Marco 1 — Captura:** Conecte um conector baseado em log à origem e transmita eventos de mudança a um tópico; inspecione os envelopes de insert/update/delete.
2. **Marco 2 — Snapshot + aplicação:** Faça um snapshot consistente, passe para streaming e aplique mudanças como upserts/deletes idempotentes no destino.
3. **Marco 3 — Ordenação, schema e lag:** Particione por chave para ordenação, trate uma mudança de schema e instrumente o lag de replicação.

## Esboço de Dados e Interface

```text
[banco de origem]  WAL / binlog
     │  captura baseada em log (Debezium)
     ▼
evento de mudança (por linha):
  { op: c|u|d, key: {id}, before: {...}, after: {...}, ts_ms, lsn }
     │  produzir ao tópico "cdc.public.orders", partição = hash(key.id)
     ▼
[log]  ordenação preservada *dentro* de uma partição (mesma chave -> mesma partição)
     ▼
[aplicação no sink]  op=c/u -> UPSERT por chave ; op=d -> DELETE por chave   (idempotente)
     linha destino: { id (pk), ..., _lsn, _updated_at }
     regra de aplicação: ignore o evento se event.lsn <= _lsn armazenado  (dedupe no replay)

Handoff snapshot->stream: snapshot no LSN X, depois stream a partir de X (sem lacuna).
Entrega: transporte at-least-once + upsert idempotente => estado effectively-once.
Métrica: lag = agora - ts_commit_origem do último evento aplicado.
```

## Desafios Extras

- Adicione tratamento de tombstones e compactação para que deletes propaguem e o tópico fique limitado.
- Suporte um contrato apoiado em schema registry e evolua uma coluna sem quebra a jusante.
- Adicione um job de reconciliação que periodicamente compara contagens/checksums de linhas origem vs destino.

## Definição de Pronto

- [ ] Inserts, updates e deletes propagam corretamente ao destino.
- [ ] O handoff snapshot-para-stream não produz lacuna nem duplicata na fronteira.
- [ ] Mudanças de mesma chave aplicam em ordem; um teste de partição embaralhada quebraria, e o seu não.
- [ ] Reproduzir de um offset antigo converge ao estado idêntico do destino (idempotência provada).
- [ ] O lag de replicação é exportado e permanece dentro do limite declarado sob uma rajada de escritas.

## Armadilhas Comuns

- CDC baseado em consulta (`WHERE updated_at > ?`) que perde deletes e hard-deletes por completo.
- Perder a ordenação particionando pela chave errada, fazendo um update chegar antes do seu insert.
- Um snapshot que não é consistente com o offset do log, deixando uma lacuna ou sobreposição no handoff.
- Aplicação não idempotente, fazendo um replay pós-crash aplicar em dobro e corromper contagens.

## Recursos

- [Documentação do Debezium](https://debezium.io/documentation/reference/stable/index.html) — a plataforma de referência de CDC baseado em log.
- [Debezium: Estrutura do evento de mudança](https://debezium.io/documentation/reference/stable/connectors/postgresql.html) — o envelope before/after/op.
- [Kafka Connect](https://kafka.apache.org/documentation/#connect) — como conectores movem eventos de CDC para um log.
- [PostgreSQL: Logical Decoding](https://www.postgresql.org/docs/current/logicaldecoding.html) — o mecanismo de WAL em que o CDC baseado em log se apoia.
