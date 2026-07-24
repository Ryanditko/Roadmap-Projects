# Interface de Chat (integração WebSocket)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um front-end de chat em tempo real que conversa com um servidor WebSocket. Diferente de um app requisição/resposta, o servidor pode enviar uma mensagem a qualquer momento, então sua UI precisa reagir a eventos que ela não pediu. Você vai abrir e gerenciar um socket, renderizar uma lista de mensagens sempre crescente que rola de forma sensata, mostrar quem está online e quem está digitando e — a parte que separa um brinquedo de um cliente de verdade — sobreviver a uma conexão perdida e reconectar com elegância. A dificuldade interessante não é desenhar balões; é manter a UI honesta sobre o estado da conexão e a ordem das mensagens quando a rede se comporta mal.

## Pré-requisitos

- Conforto com estado de componente e efeitos/ciclo de vida no seu framework (React, Vue, Svelte ou Angular)
- Entendimento de eventos e callbacks (o socket é orientado a eventos)
- Familiaridade com serialização de mensagens em JSON
- Conhecimento básico do que é um WebSocket versus HTTP

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Abrir, usar e fechar de forma limpa um WebSocket, desmontando-o quando o componente é desmontado
- Modelar o estado da conexão (conectando, aberto, reconectando, fechado) e refleti-lo na UI
- Acrescentar mensagens recebidas e manter a rolagem fixada no fundo sem prender o usuário
- Transmitir e fazer debounce de indicadores de digitação e renderizar presença
- Implementar reconexão com backoff e recuperar estado perdido na reconexão
- Tornar o log de mensagens acessível a leitores de tela conforme novo conteúdo chega

## Requisitos Funcionais

1. O app deve abrir um WebSocket ao carregar e exibir o status atual da conexão.
2. Enviar uma mensagem deve empurrá-la pelo socket e ecoá-la otimisticamente na lista.
3. Mensagens recebidas devem ser acrescentadas em ordem e renderizar remetente, texto e um timestamp formatado.
4. A lista deve rolar automaticamente para a mensagem mais recente, a menos que o usuário tenha rolado para cima para ler o histórico.
5. Um indicador de digitação deve aparecer quando outro usuário está digitando e sumir quando ele para.
6. Na desconexão, a UI deve mostrar um estado de reconexão e tentar novamente com backoff crescente.
7. O log de mensagens deve ser anunciado à tecnologia assistiva como uma região live.

## Marcos Sugeridos

1. **Marco 1 — Conectar e enviar:** Abra o socket, envie uma mensagem e renderize a lista ecoada.
2. **Marco 2 — Receber e rolar:** Trate mensagens recebidas com acréscimo ordenado e auto-rolagem inteligente.
3. **Marco 3 — Presença e digitação:** Mostre usuários online e indicadores de digitação com debounce.
4. **Marco 4 — Resiliência:** Detecte quedas, reconecte com backoff e restaure o estado.

## Esboço de Dados e Interface

```text
Layout
  [ Cabeçalho: nome da sala | status: ● conectado ]
  [ Usuários online ][ Lista de mensagens (auto-rolagem) ]
                     [ "Alice está digitando…"           ]
  [ Caixa de texto .................... | Enviar ]

Estado
  status:    'connecting' | 'open' | 'reconnecting' | 'closed'
  messages:  Message[]        (somente-acréscimo, ordenado por ts)
  users:     User[]           (presença)
  typing:    Set<userId>

Message  { id, userId, text, ts }        // ts = epoch em ms
User     { id, name, online }

Contrato WebSocket (frames JSON)
  envia -> { type: "message",  text }
           { type: "typing",   isTyping }
  recebe <- { type: "message",  message: Message }
            { type: "presence", users: User[] }
            { type: "typing",   userId, isTyping }
```

## Desafios Extras

- Adicione reações a mensagens ou confirmações de leitura.
- Persista as últimas N mensagens em `localStorage` e reidrate ao carregar.
- Agrupe mensagens consecutivas do mesmo remetente.
- Mostre um botão "N novas mensagens" quando o usuário estiver com a rolagem para cima.
- Suporte múltiplas salas com assinaturas de socket independentes.

## Definição de Pronto

- [ ] O socket abre ao carregar e fecha de forma limpa quando o componente é desmontado.
- [ ] Mensagens enviadas e recebidas aparecem na ordem correta com timestamps legíveis.
- [ ] A auto-rolagem segue novas mensagens mas cede quando o usuário rola para cima.
- [ ] Uma conexão perdida dispara um estado de reconexão visível e uma nova tentativa bem-sucedida.
- [ ] Novas mensagens são anunciadas via uma região ARIA live.

## Armadilhas Comuns

- Abrir um novo socket a cada render em vez de uma única vez, vazando conexões.
- Esquecer de fechar o socket na desmontagem, fazendo handlers dispararem contra um componente morto.
- Forçar a rolagem para o fundo a cada mensagem, arrancando o usuário do histórico que ele está lendo.
- Repetir a reconexão em um loop apertado sem backoff, martelando o servidor.
- Confiar nos relógios do cliente para a ordenação — ordene por um timestamp de servidor ou número de sequência.

## Recursos

- [MDN: A API WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) — conectando, enviando e tratando eventos.
- [MDN: Escrevendo aplicações cliente WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications) — um passo a passo prático de cliente.
- [MDN: Regiões ARIA live](https://developer.mozilla.org/pt-BR/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — anunciando conteúdo dinâmico de mensagens.
- [MDN: Intl.RelativeTimeFormat](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat) — formatando timestamps de mensagens.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — onde tempo real e APIs se situam no caminho de frontend.
