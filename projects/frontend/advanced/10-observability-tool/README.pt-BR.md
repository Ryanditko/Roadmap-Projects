# Ferramenta de Observabilidade de Frontend

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa o tipo de ferramenta que o Sentry, o Datadog RUM e o LogRocket oferecem: um SDK no cliente que captura erros, métricas de performance e comportamento do usuário de navegadores reais, envia tudo de forma confiável a um backend e o expõe em um dashboard onde você pode diagnosticar o que está quebrando e para quem. A tensão interessante é que uma ferramenta de observabilidade precisa ser quase invisível — não pode desacelerar o próprio app que mede, não pode perder dados quando a página é descarregada no meio de um relato, e deve respeitar a privacidade do usuário. Você vai projetar um SDK leve, um transporte resiliente que agrupa em lotes e sobrevive à navegação, e uma visão de agregação que transforma uma enxurrada de eventos brutos em um sinal acionável.

## Pré-requisitos

- Entendimento do modelo de erros do navegador (`error`, `unhandledrejection`) e da Performance API
- Familiaridade com os Core Web Vitals e o que cada um mede
- Conforto com batching, enfileiramento e transporte assíncrono
- Consciência das preocupações de privacidade ao coletar dados do usuário

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Capturar erros não tratados e rejeições de promise não tratadas com contexto útil (stack, breadcrumbs)
- Coletar Core Web Vitals e timings de performance customizados de usuários reais
- Enviar telemetria de forma confiável, inclusive quando a página está sendo descarregada, sem prejudicar a performance do app
- Agregar e agrupar eventos de alto volume em um dashboard diagnosticável (agrupamento de erros, tendências)
- Tratar privacidade: limpeza de PII, amostragem e respeito ao consentimento do usuário

## Requisitos Funcionais

1. Um SDK deve capturar erros não tratados e rejeições não tratadas com stack traces e breadcrumbs contextuais.
2. O SDK deve coletar Core Web Vitals (LCP, INP, CLS) e permitir marcas de timing customizadas.
3. A telemetria deve ser agrupada em lotes e enviada sem bloquear a thread principal ou degradar a performance do app.
4. Dados enfileirados no descarregamento da página ainda devem ser entregues (ex.: via um mecanismo keep-alive/beacon).
5. Erros similares devem ser agrupados (fingerprinted) para que um pico seja um problema, não milhares de linhas.
6. Um dashboard deve mostrar a frequência de erros, sessões afetadas e tendências de performance ao longo do tempo.
7. PII deve ser removível e a coleta deve honrar uma configuração de consentimento/amostragem.

## Marcos Sugeridos

1. **Marco 1 — SDK de captura:** Conecte handlers de erro e rejeição; registre breadcrumbs e contexto.
2. **Marco 2 — Performance e transporte:** Colete Web Vitals e construa um transporte em lotes e seguro no descarregamento.
3. **Marco 3 — Ingestão e agrupamento:** Armazene eventos no servidor e faça fingerprint dos erros em issues agrupados.
4. **Marco 4 — Dashboard e privacidade:** Construa a visão de tendências/issues e adicione limpeza de PII + amostragem.

## Esboço de Dados e Interface

```text
   Navegador (app instrumentado)
   ┌──────────────────────────────────────────────┐
   │ SDK                                            │
   │  window.onerror / onunhandledrejection ──▶ evento│
   │  PerformanceObserver (LCP/INP/CLS)      ──▶ evento│
   │  breadcrumbs (cliques, navegações, fetches)     │
   │           │ limpa PII · amostra                 │
   │           ▼                                     │
   │  fila em lote ──flush por: tamanho | intervalo | unload│
   └───────────┬────────────────────────────────────┘
               │ navigator.sendBeacon / fetch keepalive
               ▼
   API de ingestão ─▶ armazena ─▶ fingerprint e agrupa ─▶ Dashboard
                                                          (issues, tendências, sessões)

Evento:  { type: error|vital|breadcrumb, ts, session, release, payload }
Fingerprint:  hash(mensagem normalizada + frames de topo da stack)
Metas não funcionais:
  bundle do SDK       <= 15 KB gzipado
  custo na thread     desprezível (sem tarefas longas do SDK)
  entrega no unload   nenhum dado perdido quando a aba fecha
  privacidade         PII limpa antes do envio · amostragem configurável
```

## Desafios Extras

- Adicione um session replay-lite: uma linha do tempo de breadcrumbs reconstruindo os passos antes de um erro.
- Adicione alertas quando a taxa de um erro cruza um limiar, com um hook de notificação.
- Adicione saúde de release: compare taxas de erro entre versões do app para pegar um deploy ruim.
- Adicione suporte a source-maps para que stack traces minificados resolvam para o código original.

## Definição de Pronto

- [ ] Um erro lançado e uma promise rejeitada aparecem no dashboard com stack e breadcrumbs.
- [ ] Core Web Vitals de um carregamento de página real são capturados e plotados.
- [ ] Eventos enfileirados logo antes de um descarregamento de página ainda são entregues.
- [ ] Milhares de erros idênticos colapsam em um único issue agrupado com uma contagem.
- [ ] PII é removida antes da transmissão e a amostragem reduz o volume conforme configurado.

## Armadilhas Comuns

- Enviar cada evento imediatamente, martelando a rede e desacelerando o app que mede.
- Usar `fetch` sem `keepalive` no descarregamento, de modo que os últimos (frequentemente mais importantes) eventos são perdidos.
- Fazer fingerprint na mensagem de erro bruta, de modo que valores dinâmicos (ids, URLs) explodem um issue em milhares.
- Capturar todo o DOM ou o conteúdo de formulários como breadcrumbs, vazando senhas e dados pessoais.
- Instrumentar de forma síncrona em caminhos quentes, adicionando tarefas longas que distorcem as próprias métricas que você coleta.

## Recursos

- [MDN: evento error da Window](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/error_event) — capturando erros não tratados no navegador.
- [web.dev: Measure performance with the RUM Data Model](https://web.dev/articles/vitals-measurement-getting-started) — coletando Web Vitals de usuários reais.
- [MDN: Navigator.sendBeacon()](https://developer.mozilla.org/pt-BR/docs/Web/API/Navigator/sendBeacon) — enviando dados de forma confiável quando a página descarrega.
- [MDN: PerformanceObserver](https://developer.mozilla.org/pt-BR/docs/Web/API/PerformanceObserver) — observando entradas de performance sem polling.
