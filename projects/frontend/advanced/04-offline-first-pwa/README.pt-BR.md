# PWA Offline-First

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um Progressive Web App que trata a rede como um aprimoramento, não um requisito — o app continua funcionando em um trem, num túnel ou numa conexão instável, e depois sincroniza discretamente quando a rede volta. Isso inverte a suposição usual de que os dados vivem em um servidor e o cliente é uma visão fina. Aqui o cliente é dono de um armazenamento local durável, renderiza a partir dele instantaneamente e reconcilia com o backend em segundo plano. As partes difíceis são as que os usuários só notam quando quebram: um cache obsoleto servindo dados da semana passada, um background sync que perde silenciosamente uma escrita, ou duas edições que colidem após uma reconexão. Você vai projetar estratégias de cache de forma deliberada e tornar o estado offline uma parte visível e de primeira classe da UX.

## Pré-requisitos

- Uma SPA funcional que você possa adaptar (um projeto [Intermediário](../../intermediate/) funciona bem)
- Entendimento do ciclo de vida requisição/resposta e dos cabeçalhos de cache HTTP
- Familiaridade com Promises e fluxo de dados assíncrono
- Consciência do IndexedDB ou de um wrapper como banco de dados no cliente

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Registrar um service worker e interceptar requisições de rede com estratégias de cache escolhidas
- Escolher estratégias por recurso (cache-first, network-first, stale-while-revalidate) e justificar cada uma
- Persistir dados da aplicação localmente no IndexedDB e renderizar a partir deles antes de a rede responder
- Enfileirar escritas feitas offline e reproduzi-las de forma confiável na reconexão via Background Sync
- Comunicar o status de conectividade e de sincronização com clareza para que o usuário nunca fique confuso sobre a atualidade dos dados

## Requisitos Funcionais

1. O app deve carregar e ser utilizável em uma visita repetida com a rede totalmente desativada.
2. Os assets estáticos (app shell) devem ser servidos do cache e atualizados com segurança quando uma nova versão for implantada.
3. Os dados da aplicação devem ser legíveis offline a partir de um armazenamento local, não apenas HTML estático.
4. Uma escrita feita offline deve ser enfileirada e sincronizada automaticamente assim que a conectividade retornar.
5. A UI deve indicar claramente o status offline e as mudanças pendentes, não sincronizadas.
6. Uma nova versão do service worker não deve servir uma mistura quebrada de assets antigos e novos.
7. Edições conflitantes (local vs. servidor) devem ser resolvidas por uma estratégia explícita e documentada — não por perda silenciosa.

## Marcos Sugeridos

1. **Marco 1 — Shell instalável:** Adicione um manifest e um service worker que cacheia o app shell para carga offline.
2. **Marco 2 — Dados offline:** Armazene e leia dados da aplicação no IndexedDB; renderize a partir deles primeiro.
3. **Marco 3 — Fila de escrita e sync:** Enfileire mutações offline e reproduza-as com Background Sync na reconexão.
4. **Marco 4 — Conflitos e atualizações:** Trate conflitos de edição e trocas seguras de versão do service worker.

## Esboço de Dados e Interface

```text
   Aba do navegador            Service worker              Rede
 ┌────────────┐   fetch      ┌─────────────────┐   fetch   ┌────────┐
 │  UI lê do  │────────────▶ │  roteador de     │─────────▶ │  API   │
 │  IndexedDB │◀──────────── │  estratégia      │◀───────── │        │
 └─────┬──────┘   resposta   └────────┬────────┘           └────────┘
       │ escritas                      │ cache
 ┌─────▼───────────┐         ┌─────────▼────────┐
 │  IndexedDB       │         │  Cache Storage    │
 │  dados + outbox  │         │  app shell + assets│
 └─────┬───────────┘         └──────────────────┘
       │ Background Sync reproduz a outbox na reconexão
       └──────────────────────────────────────────▶ API

Estratégias:  shell=cache-first · dados=stale-while-revalidate · escritas=enfileiradas
Conflitos:    last-write-wins | vetor de versão | merge manual — escolha + documente

Metas não funcionais:
  visita repetida offline   totalmente utilizável
  tamanho do shell cacheado <= 200 KB
  escrita enfileirada na reconexão  nunca perdida, reproduzida uma vez (idempotente)
```

## Desafios Extras

- Adicione notificações push que aparecem mesmo com o app fechado.
- Implemente sincronização periódica em segundo plano para atualizar dados antes de o usuário reabrir o app.
- Adicione um prompt de instalação com uma UX customizada e bem cronometrada, em vez do banner cru do navegador.
- Mostre um selo de sync por item (sincronizado / pendente / falhou) com nova tentativa.

## Definição de Pronto

- [ ] Com a rede desligada, um visitante recorrente consegue abrir o app, ler dados e fazer uma edição.
- [ ] Essa edição offline aparece sincronizada ao servidor automaticamente após reconectar, exatamente uma vez.
- [ ] Implantar uma nova versão atualiza o service worker sem servir um conjunto de assets incompatível.
- [ ] A UI mostra o status offline e uma contagem de mudanças não sincronizadas.
- [ ] Um conflito de edição criado deliberadamente resolve pela estratégia documentada, sem perder intenção do usuário silenciosamente.

## Armadilhas Comuns

- Cachear tudo com cache-first, deixando usuários presos em dados obsoletos sem caminho de atualização.
- Esquecer o ciclo de vida do service worker (`waiting`/`skipWaiting`), deixando usuários em uma versão antiga indefinidamente.
- Tratar escritas no IndexedDB como síncronas, causando updates perdidos sob interação rápida.
- Reproduzir a outbox offline sem idempotência, duplicando registros no servidor em reconexões instáveis.
- Esconder o estado offline por completo, fazendo usuários pensarem que seu trabalho não sincronizado está salvo no servidor.

## Recursos

- [web.dev: Offline cookbook](https://web.dev/articles/offline-cookbook) — o catálogo definitivo de estratégias de cache de service worker.
- [MDN: Service Worker API](https://developer.mozilla.org/pt-BR/docs/Web/API/Service_Worker_API) — ciclo de vida, escopo e interceptação de fetch.
- [MDN: IndexedDB API](https://developer.mozilla.org/pt-BR/docs/Web/API/IndexedDB_API) — o banco de dados durável do navegador no cliente.
- [web.dev: Background Sync](https://web.dev/articles/background-sync) — adiar escritas de forma confiável até a conectividade retornar.
