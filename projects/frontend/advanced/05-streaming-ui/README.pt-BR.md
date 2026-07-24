# UI de Streaming (plataforma de vídeo)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa a experiência de reprodução de uma plataforma de vídeo como o YouTube ou a Netflix — streaming adaptativo que muda a qualidade conforme a rede, controles fluidos e uma interface que permanece responsiva enquanto um pipeline de mídia pesado roda por baixo. A percepção central é que você não baixa um vídeo; você baixa um manifesto que descreve muitas renderizações de qualidade divididas em pequenos segmentos, e o player escolhe continuamente qual segmento buscar em seguida com base na largura de banda medida e na saúde do buffer. Erre essa adaptação e o usuário vê travadas ou quadros borrados; erre a UI e os controles ficam lentos contra o trabalho pesado de decodificação. Este projeto é sobre orquestrar um motor de bitrate adaptativo e um player polido e acessível ao redor dele.

## Pré-requisitos

- Confiança com fluxo de dados assíncrono e eventos do navegador
- Entendimento de requisições HTTP de intervalo (range) e conceitos de buffering
- Familiaridade com o elemento HTML5 `<video>` e Media Source Extensions (ao menos conceitualmente)
- Noção básica de vazão de rede e de como ela varia

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Explicar o streaming de bitrate adaptativo (HLS/DASH): manifestos, renderizações e segmentos
- Integrar um motor ABR e raciocinar sobre suas heurísticas de seleção de qualidade
- Construir controles de reprodução acessíveis e operáveis por teclado que permanecem responsivos sob carga
- Gerenciar o buffering para minimizar travadas evitando uso excessivo de memória
- Instrumentar a qualidade de reprodução (travadas, tempo de início, bitrate) para uma visão de qualidade de experiência

## Requisitos Funcionais

1. O player deve reproduzir um stream adaptativo (HLS ou DASH) e trocar renderizações automaticamente conforme a largura de banda muda.
2. Um usuário deve poder sobrescrever a qualidade automática e fixar uma renderização específica.
3. Os controles de reprodução (play/pause, buscar, volume, tela cheia) devem ser totalmente operáveis por teclado e rotulados.
4. Buscar para qualquer ponto deve fazer buffer e retomar sem recarregar o stream inteiro.
5. O player deve se recuperar com elegância de uma queda de rede transitória sem um erro fatal.
6. A memória de buffer deve ser limitada para que sessões longas não cresçam indefinidamente.
7. As métricas de qualidade de reprodução (tempo de início, contagem de travadas, bitrate atual) devem ser observáveis.

## Marcos Sugeridos

1. **Marco 1 — Reprodução adaptativa básica:** Carregue um manifesto e reproduza com um motor ABR, apenas qualidade automática.
2. **Marco 2 — Controles e a11y:** Construa controles customizados acessíveis: buscar, volume, tela cheia, legendas.
3. **Marco 3 — Qualidade e resiliência:** Adicione sobrescrita manual de qualidade e recuperação de interrupções de rede.
4. **Marco 4 — Instrumentação de QoE:** Rastreie travadas, tempo de início e bitrate; exiba-os em um overlay de debug.

## Esboço de Dados e Interface

```text
   Manifesto (.m3u8 / .mpd)
     ├── renderização 240p  → seg0 seg1 seg2 ...
     ├── renderização 480p  → seg0 seg1 seg2 ...
     └── renderização 1080p → seg0 seg1 seg2 ...
                    │
                    ▼
        ┌───────────────────────────┐
        │   motor ABR (hls.js/dash)  │  mede largura de banda + buffer
        │   escolhe próximo segmento │─────────────┐
        └─────────────┬─────────────┘             │ anexa ao
                      │                             ▼
                      │                    ┌──────────────────┐
                      │                    │ buffer MediaSource│→ <video>
        ┌─────────────▼─────────────┐      └──────────────────┘
        │  UI do player (controles)  │
        │  play/buscar/volume/qual.  │
        └────────────────────────────┘

Métricas de QoE:  startupTime, rebufferCount, currentBitrate, droppedFrames
Metas não funcionais:
  tempo de início     < 2 s em banda larga
  razão de rebuffer   < 1% do tempo assistido
  resposta do controle < 100 ms independente da carga de decode
```

## Desafios Extras

- Adicione legendas (WebVTT) com um seletor de faixas e controles de estilo.
- Adicione picture-in-picture e um mini-player que persiste através da navegação.
- Implemente uma tira de prévia de miniaturas na barra de busca (baseada em sprite).
- Adicione um recurso de histórico/retomada que restaura a última posição.

## Definição de Pronto

- [ ] O stream visivelmente reduz a qualidade em uma conexão limitada e se recupera quando a largura de banda retorna.
- [ ] Todos os controles funcionam apenas com teclado e anunciam o estado a um leitor de tela.
- [ ] Buscar no meio do stream retoma rapidamente sem uma recarga completa.
- [ ] Uma oscilação de rede simulada não lança um erro fatal; a reprodução retoma.
- [ ] O overlay de QoE reporta tempo de início, contagem de travadas e bitrate atual ao vivo.

## Armadilhas Comuns

- Fazer seu próprio buscador de segmentos em vez de um motor ABR comprovado — as heurísticas são difíceis de acertar.
- Construir controles com elementos não interativos, quebrando o acesso por teclado e leitor de tela.
- Nunca liberar segmentos em buffer, fazendo a memória subir ao longo de uma sessão longa de visualização.
- Tratar um 5xx transitório em um segmento como fatal em vez de tentar o próximo.
- Medir a largura de banda apenas no início, de modo que a qualidade nunca se adapta a uma mudança de rede no meio do stream.

## Recursos

- [MDN: Media Source Extensions API](https://developer.mozilla.org/pt-BR/docs/Web/API/Media_Source_Extensions_API) — a API do navegador por trás do streaming adaptativo.
- [Documentação do hls.js](https://github.com/video-dev/hls.js/#documentation) — um motor de reprodução HLS amplamente usado na web.
- [web.dev: Fast playback with audio and video preload](https://web.dev/articles/fast-playback-with-preload) — reduzindo a latência de início.
- [MDN: WebVTT](https://developer.mozilla.org/pt-BR/docs/Web/API/WebVTT_API) — o padrão para legendas e closed captions.
