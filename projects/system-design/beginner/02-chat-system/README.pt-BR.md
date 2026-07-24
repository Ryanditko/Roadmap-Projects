# Projete um Sistema de Chat em Tempo Real

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um sistema de chat um-para-um e de pequenos grupos — pense em um WhatsApp ou DM do Slack enxuto. O cerne do problema é que uma mensagem precisa viajar de um cliente conectado a outro quase em tempo real, o que significa que apenas requisição/resposta HTTP não basta. Você vai raciocinar sobre conexões persistentes, como mensagens são persistidas e ordenadas, e como um usuário sabe quem está online. Entregue um documento de design que explique entrega, armazenamento e presença — não um servidor funcional.

## Pré-requisitos

- Entendimento do modelo cliente/servidor e conexões TCP
- Consciência de que WebSocket mantém uma conexão aberta, ao contrário do HTTP comum
- Familiaridade com filas e como uma mensagem pode ser bufferizada
- Conforto para ler uma sequência de interações entre componentes

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher um transporte em tempo real (WebSocket vs. long polling) e justificá-lo
- Projetar um modelo de mensagem que suporte ordenação e status de entrega
- Explicar como mensagens offline são armazenadas e entregues depois
- Rastrear presença de usuários e raciocinar sobre sua precisão
- Enunciar um trade-off entre garantias de entrega e complexidade

## Requisitos e Restrições

1. Dois usuários online trocam mensagens com entrega em menos de um segundo.
2. Mensagens enviadas enquanto o destinatário está offline são armazenadas e entregues ao reconectar.
3. Mensagens dentro de uma conversa devem aparecer em ordem consistente.
4. Mostrar se um contato está online ou offline (presença).
5. Suportar ao menos chat 1-para-1; chat em grupo é um desafio extra.
6. Estime a escala: 100K usuários ativos diários, média de 40 mensagens/usuário/dia.

## Abordagem Sugerida

1. Separe o transporte (como os bytes se movem) da persistência (como as mensagens são armazenadas).
2. Faça a conta: 100K × 40 = 4M mensagens/dia ≈ 46 mensagens/s em média, com picos várias vezes maiores.
3. Decida como um cliente mantém uma conexão e com qual servidor ele fala.
4. Projete o armazenamento de mensagens com uma chave de conversa e uma sort key ordenada por tempo.
5. Adicione um mecanismo de presença (heartbeat + last-seen) e descreva sua janela de defasagem.

## Esboço de Arquitetura

```text
Cliente A <--WebSocket--> [ Gateway ] --> [ Serviço de Mensagens ] --> [ Store de Mensagens ]
Cliente B <--WebSocket--> [ Gateway ]              │
                                             [ Presença ]  (heartbeat, last_seen)
                          offline? enfileira -> [ Inbox / Fila ] -> entrega ao reconectar

API / eventos principais
  WS send   { conversationId, body }        -> ack { messageId, ts }
  WS deliver{ messageId, from, body, ts }
  GET /conversations/{id}/messages?before=ts -> página de mensagens

Modelo de dados
  messages: conversation_id (PK) | ts (SK) | sender_id | body | status
  presence: user_id | last_seen | online
```

## Tópicos de Aprofundamento

- **Garantias de entrega:** at-least-once vs. exactly-once, e como IDs de mensagem permitem dedup no cliente.
- **Ordenação:** timestamps atribuídos pelo servidor vs. números de sequência por conversa.
- **Roteamento de conexão:** como um gateway mapeia um usuário online ao servidor que segura seu socket.

## Entregáveis

- Um diagrama de arquitetura mostrando transporte, armazenamento e presença.
- O contrato de envio/entrega de mensagens e um endpoint de busca de histórico.
- Um modelo de dados para mensagens e presença.
- Um trade-off descrito: ex., WebSocket (tempo real de verdade, servidores com estado, mais difícil de escalar) vs. long polling (mais simples, maior latência e overhead).

## Armadilhas Comuns

- Assumir que requisição/resposta HTTP comum entrega mensagens no instante em que são enviadas.
- Ignorar a ordenação, fazendo mensagens aparecerem fora de sequência sob concorrência.
- Esquecer o caso offline — mensagens enviadas a um usuário desconectado não podem sumir.
- Tratar presença como exata; heartbeats sempre deixam uma janela de defasagem.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões para sistemas de tempo real e mensageria.
- [MDN: API WebSocket](https://developer.mozilla.org/pt-BR/docs/Web/API/WebSockets_API) — como funcionam conexões persistentes.
- [RFC 6455: O Protocolo WebSocket](https://datatracker.ietf.org/doc/html/rfc6455) — a especificação do protocolo.
- [High Scalability: arquitetura do WhatsApp](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html) — um sistema real de mensageria em escala.
