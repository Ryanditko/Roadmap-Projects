# Projete um Encurtador de URLs

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um serviço como o Bitly que transforma uma URL longa em um código curto e redireciona os visitantes ao original. É o clássico primeiro problema de system design porque força você a raciocinar sobre tráfego pesado de leitura, geração de chaves únicas e onde colocar um cache — tudo sem muita lógica de negócio no caminho. Aqui seu objetivo é produzir um documento de design, não um serviço rodando: decida como os códigos são gerados, como o mapeamento é armazenado e como um redirecionamento se mantém rápido quando as leituras superam as escritas em cem para um.

## Pré-requisitos

- Um modelo mental de como funcionam requisições HTTP e redirecionamentos 3xx
- Familiaridade com a diferença entre um banco relacional e um chave-valor
- Ter construído a versão em código ajuda ([Encurtador de URLs (Em Memória)](../../../backend/beginner/02-url-shortener-in-memory/))
- Conforto para ler um diagrama simples de caixas e setas

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Separar requisitos funcionais dos não-funcionais (latência, disponibilidade)
- Fazer contas de guardanapo para dimensionar armazenamento e QPS
- Comparar estratégias de geração de código curto e seu comportamento em colisões
- Justificar onde posicionar um cache em um caminho pesado de leitura
- Articular por escrito um trade-off concreto

## Requisitos e Restrições

1. Encurtar uma URL `http`/`https` válida em um código único; rejeitar entrada inválida.
2. Redirecionar um código curto à URL original com baixa latência (meta p99 < 100 ms).
3. Leituras superam muito as escritas — assuma uma proporção 100:1 de leitura/escrita.
4. Códigos devem ser curtos (6–8 caracteres) e seguros para URL.
5. O sistema deve permanecer disponível para redirecionamentos mesmo durante falhas de escrita.
6. Estime a escala: ~10M novos links/mês, ~1B redirecionamentos/mês.

## Abordagem Sugerida

1. Escreva o caminho de leitura e o de escrita separadamente — eles têm necessidades diferentes.
2. Faça a estimativa: 10M escritas/mês ≈ 4 escritas/s; 1B leituras/mês ≈ 400 leituras/s. Armazenamento: 10M × ~500 bytes ≈ 5 GB/mês.
3. Escolha uma estratégia de código — base62 aleatório com verificação de unicidade, ou um contador codificado em base62.
4. Defina o banco de dados e, então, coloque um cache na frente do caminho de leitura.
5. Explique como um cache miss é tratado e como o cache é populado.

## Esboço de Arquitetura

```text
Cliente ── POST /shorten ──> [ App ] ──> [ Store Primário ]
Cliente ── GET /{code} ────> [ App ] ──> [ Cache ] --miss--> [ Store ]
                                              │
                                         302 Location: longUrl

API principal
  POST /shorten   { url }        -> 201 { code, shortUrl }
  GET  /{code}                   -> 302 Location: <longUrl> | 404

Modelo de dados
  links: code (PK) | long_url | created_at | hits
  counter (opcional): linha/sequência atômica única para codificação base62
```

## Tópicos de Aprofundamento

- **Geração de chaves:** probabilidade de colisão de códigos aleatórios vs. o custo de coordenação de um contador global.
- **Cache:** eviction por LRU vs. TTL; qual taxa de acerto de cache você precisa para atingir a meta de latência.
- **Disponibilidade:** por que redirecionamentos podem ser servidos de uma réplica de leitura ou do cache durante uma falha do primário.

## Entregáveis

- Um diagrama de arquitetura mostrando os caminhos de leitura e escrita.
- O contrato da API principal (endpoints, métodos, códigos de status).
- Um modelo de dados para o mapeamento de links.
- Um trade-off chave descrito: ex., códigos aleatórios (sem coordenação, exigem retry em colisão) vs. códigos baseados em contador (previsíveis, mas gargalo de coordenação).

## Armadilhas Comuns

- Detalhar o caminho de escrita e ignorar o de leitura, que carrega 99% do tráfego.
- Escolher um comprimento de código sem calcular quantas URLs ele pode representar.
- Adicionar um cache sem definir a política de eviction ou como os misses são tratados.
- Tratar "código único" como grátis — toda estratégia tem um custo de colisão ou de coordenação.

## Recursos

- [System Design Primer: Encurtador de URLs](https://github.com/donnemartin/system-design-primer#design-a-url-shortening-service-like-bitly) — um exemplo completo resolvido.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — o guia de estudos open-source canônico.
- [MDN: Redirecionamentos HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Redirections) — 301 vs 302 e comportamento de cache.
- [Wikipedia: Base62](https://en.wikipedia.org/wiki/Base62) — a codificação padrão para códigos curtos.
