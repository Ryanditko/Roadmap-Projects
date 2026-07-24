# Backend de Chat em Tempo Real (WebSockets)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa o backend por trás de um app de chat como o Slack ou o WhatsApp Web: mensagens aparecem no instante em que são enviadas, você vê quem está online e uma conexão que caiu se recupera sem perder nada. O núcleo é um servidor WebSocket segurando milhares de conexões de longa duração, mas a parte difícil é o que acontece quando você extrapola um único processo. Dois usuários conectados a instâncias diferentes do servidor ainda precisam ver as mensagens um do outro, então você introduzirá uma espinha dorsal de pub/sub e enfrentará as perguntas que definem sistemas de tempo real: como distribuir (fan-out) uma mensagem para os sockets certos, como manter as mensagens em ordem e como tornar a presença precisa quando conexões surgem e somem silenciosamente.

## Pré-requisitos

- Confiança para construir APIs HTTP e raciocinar sobre o ciclo requisição/resposta
- Entendimento do handshake WebSocket e de como ele faz upgrade a partir do HTTP
- Familiaridade com um event loop / I/O assíncrono na sua linguagem de escolha
- Exposição básica a um pub/sub ou message broker (Redis Pub/Sub, NATS, Kafka)
- Uma stack de sua escolha (Node, Go, Elixir, Python) além de um datastore para o histórico de mensagens

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Gerenciar o ciclo de vida de milhares de conexões WebSocket persistentes
- Distribuir mensagens (fan-out) entre múltiplas instâncias do servidor via uma espinha dorsal de pub/sub
- Raciocinar sobre ordenação de mensagens e garantias de entrega (at-least-once vs. exactly-once)
- Rastrear presença com precisão usando heartbeats e detectando desconexões silenciosas
- Bufferizar e reprocessar mensagens perdidas para que um cliente reconectando se atualize
- Aplicar backpressure para que um consumidor lento não esgote a memória do servidor

## Requisitos Funcionais

1. Clientes devem conectar por WebSocket, autenticar na conexão e entrar em uma ou mais salas.
2. Uma mensagem enviada a uma sala deve ser entregue a cada membro conectado, incluindo os em outras instâncias do servidor.
3. Mensagens dentro de uma sala devem ser entregues em uma ordem consistente para todos os destinatários.
4. O sistema deve rastrear presença (online/offline/digitando) e transmitir mudanças aos clientes interessados.
5. Deve persistir o histórico de mensagens para que um cliente possa carregar mensagens anteriores e retomar após reconectar.
6. Deve detectar conexões mortas via heartbeat/ping-pong e limpar seus recursos.
7. **Escalabilidade:** o design deve escalar horizontalmente — adicionar instâncias deve aumentar a capacidade de conexões sem assumir estado compartilhado no processo.
8. **Confiabilidade:** uma mensagem aceita pelo servidor não deve ser silenciosamente perdida se um destino de entrega estiver brevemente offline.
9. **Modos de falha:** o crash de uma instância não deve derrubar as conexões de outras instâncias; clientes reconectam e ressincronizam.

## Marcos Sugeridos

1. **Marco 1 — Chat em nó único:** Um servidor, salas em memória, broadcast para os sockets conectados.
2. **Marco 2 — Escalar horizontalmente:** Adicione uma espinha dorsal de pub/sub para que mensagens cruzem instâncias; torne os servidores stateless.
3. **Marco 3 — Persistência e retomada:** Armazene histórico, atribua números de sequência, reprocesse na reconexão.
4. **Marco 4 — Presença e resiliência:** Heartbeats, indicadores de digitação, backpressure e shutdown gracioso.

## Esboço de Dados e Interface

```text
Visão de componentes

   clientes ==ws==> [Instância A] --publish--> [ Barramento Pub/Sub ] <--publish-- [Instância B] <==ws== clientes
                         |                              |                                |
                         +--> [ Store de Mensagens ] <---+---- sequência por sala --------+
                         +--> [ Store de Presença (chaves TTL renovadas por heartbeat) ]

Frames cliente -> servidor
  { type: "join",    room }
  { type: "message", room, body, clientMsgId }
  { type: "typing",  room }
  { type: "ping" }

Frames servidor -> cliente
  { type: "message", room, seq, senderId, body, ts }
  { type: "presence", room, userId, status }
  { type: "ack", clientMsgId, seq }
  { type: "pong" }

Reconexão: cliente envia último seq visto por sala -> servidor reprocessa a lacuna do store
```

## Desafios Extras

- Adicione confirmações de leitura e reações de mensagem com fan-out correto para o remetente.
- Suporte busca de mensagens sobre o histórico com um índice dedicado.
- Adicione encriptação ponta a ponta ou em repouso para os corpos das mensagens.
- Introduza moderação: rate limits, silenciamento e filtro de palavrões.

## Definição de Pronto

- [ ] Dois clientes em instâncias diferentes do servidor trocam mensagens em tempo real.
- [ ] Mensagens em uma sala chegam na mesma ordem para todos os destinatários.
- [ ] Um cliente que desconecta e reconecta recebe cada mensagem que perdeu, exatamente uma vez.
- [ ] A presença reflete a realidade em segundos, e desconexões silenciosas são detectadas via heartbeat.
- [ ] Matar uma instância não derruba conexões nas outras; clientes ressincronizam automaticamente.

## Armadilhas Comuns

- Assumir um único processo: o estado de sala em memória quebra no momento em que você adiciona uma segunda instância.
- Confiar no TCP para notar uma conexão morta — sockets meio-abertos exigem heartbeats no nível da aplicação.
- Ignorar backpressure, de modo que o buffer de envio ilimitado de um cliente lento acaba derrubando o servidor.
- Usar timestamps para ordenação entre instâncias; a defasagem de relógio reordena mensagens — use sequências por sala.
- Transmitir presença a cada tecla, inundando o barramento — faça debounce e agrupe eventos de digitação.

## Recursos

- [MDN: A API WebSocket](https://developer.mozilla.org/pt-BR/docs/Web/API/WebSockets_API) — protocolo e básicos do cliente.
- [RFC 6455: O Protocolo WebSocket](https://datatracker.ietf.org/doc/html/rfc6455) — a spec autoritativa, incluindo ping/pong.
- [Redis: Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — uma espinha dorsal comum de fan-out entre instâncias.
- [O problema C10K](http://www.kegel.com/c10k.html) — o ensaio clássico sobre lidar com muitas conexões concorrentes.
- [Ably: WebSockets vs. long polling](https://ably.com/topic/websockets-vs-long-polling) — trade-offs e contexto de escala.
