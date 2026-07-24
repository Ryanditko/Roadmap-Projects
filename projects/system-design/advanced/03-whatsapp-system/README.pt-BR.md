# Projete uma Plataforma de Mensagens como o WhatsApp

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete uma plataforma de mensagens mobile-first no estilo do WhatsApp: bilhões de usuários trocam mensagens individuais e em grupo que precisam chegar exatamente uma vez, em ordem, com confirmações de entrega e leitura — mesmo quando o destinatário está offline por horas. O desafio central é manter centenas de milhões de conexões persistentes, rotear cada mensagem para o fan-out de dispositivo correto e fazer tudo isso sob criptografia ponta a ponta, onde o servidor nunca vê o texto puro. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Entendimento sólido de TCP, WebSockets e gerenciamento de conexões de longa duração
- Familiaridade com filas de mensagens e entrega at-least-once vs exactly-once
- Criptografia básica: chaves simétricas/assimétricas, sigilo futuro, o protocolo Signal
- Entendimento de sistemas de push notification (APNs, FCM)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma camada de conexão que mantém centenas de milhões de sessões concorrentes
- Garantir entrega ordenada e exactly-once sobre um transporte at-least-once
- Modelar entrega offline com uma caixa de entrada durável por destinatário e confirmações
- Raciocinar sobre criptografia ponta a ponta e por que o servidor armazena só texto cifrado
- Projetar fan-out de grupo que escala sem multiplicação quadrática de mensagens

## Requisitos e Restrições

1. Entregar mensagens com ordenação por conversa e semântica exactly-once.
2. Suportar destinatários offline: armazenar e encaminhar quando reconectarem.
3. Manter ~500M de conexões concorrentes; assumir dezenas de bilhões de mensagens/dia.
4. Fornecer confirmações enviado/entregue/lido com round-trips extras mínimos.
5. Aplicar criptografia ponta a ponta; servidores nunca devem ter texto puro ou chaves de longo prazo.
6. Suportar grupos de até ~1024 membros sem explosão quadrática.
7. Mirar 99,99% de disponibilidade com entrega sub-segundo para usuários online.

## Abordagem Sugerida

Estime primeiro o fan-out de conexão: sessões concorrentes por nó de gateway determina o tamanho da frota e a memória. Divida o sistema em uma **camada de conexão** (quase sem estado, gateways segurando sockets) e uma **camada de roteamento** que mapeia usuário→gateway ativo via um registro de presença. Persista mensagens não entregues em uma caixa de entrada por usuário chaveada para ordenação, apagando no ack. Atribua números de sequência monotônicos por conversa para ordenação e faça dedup no cliente. Mantenha a criptografia E2E no lado do cliente (double ratchet estilo Signal): o servidor roteia blobs opacos de texto cifrado. Para grupos, cifre uma vez por chave de remetente e faça fan-out de referências em vez de recifrar por membro.

## Esboço de Arquitetura

```text
Fone A ──WSS──> Gateway (segura socket) ──> Router ──> Registro de presença (user->gateway)
                                              │
                                              ├──> Store de mensagens (inbox por usuário, durável)
                                              └──> Push (APNs/FCM) se offline

Fluxo de envio:
A cifra (double ratchet) -> Gateway -> Router -> B online? entrega : enfileira inbox + push
B reconecta -> drena inbox (ordenado) -> ack -> servidor apaga -> confirmações voltam para A

APIs principais (sobre socket persistente, framed):
SEND    { convId, seq, ciphertext, ts }        -> ACK { convId, seq }
RECEIPT { convId, seq, type: DELIVERED|READ }
PRESENCE{ userId, state: ONLINE|OFFLINE, lastSeen }

Modelo de dados (esboço):
Inbox{ userId, [ {convId, seq, ciphertext, enqueuedAt} ] }  # apagado no ack
Session{ userId, deviceId, gatewayNode, prekeys[] }
Group{ id, members[], senderKeys{} }  # fan-out por referência
```

## Tópicos de Aprofundamento

- **Escala da camada de conexão:** event loops/epoll, orçamento de sockets por nó, failover gracioso de gateway.
- **Ordenação e dedup:** números de sequência por conversa, aplicação idempotente no cliente.
- **Store-and-forward offline:** durabilidade da inbox, ack-e-apaga, limites de retenção.
- **Criptografia ponta a ponta:** double ratchet, prekeys, sincronização de chaves multi-dispositivo.
- **Fan-out de grupo:** sender keys vs criptografia par a par; limites de amplificação em grupos grandes.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com o fluxo de envio/recebimento e modelo de inbox, refinado.
- [ ] Estimativas de capacidade para conexões concorrentes, frota de gateways e volume diário de mensagens.
- [ ] O mecanismo de ordenação e exactly-once detalhado de ponta a ponta.
- [ ] Uma análise de falha/DR: crash de gateway, perda do registro de presença, falha de partição da inbox.
- [ ] Uma seção de criptografia E2E explicando o que o servidor pode e não pode ver.

## Armadilhas Comuns

- Tratar o servidor como confiável com texto puro — E2E significa que o servidor roteia só texto cifrado.
- Confiar na ordenação do transporte; adicione números de sequência explícitos por conversa.
- Nunca apagar a inbox após o ack, fazendo o armazenamento crescer sem limite.
- Fan-out de grupo que recifra por membro, transformando um envio em milhares de operações de cripto.
- Perder o registro de presença sem fallback para rerotear usuários reconectando.

## Recursos

- [O Protocolo Signal](https://signal.org/docs/) — o double-ratchet E2E em que o WhatsApp se baseia.
- [High Scalability: arquitetura do WhatsApp](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html) — como um time minúsculo manteve milhões de conexões.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — mensageria, filas e escala de conexões.
- [RFC 6455: O Protocolo WebSocket](https://datatracker.ietf.org/doc/html/rfc6455) — o transporte de conexão persistente.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — garantias de entrega e ordenação.
