# Projete um Balanceador de Carga Global

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete a camada de direcionamento de tráfego que fica na frente de um serviço distribuído globalmente e decide, para cada requisição de entrada, qual datacenter deve atendê-la. Um bom balanceador de carga global roteia usuários para a região saudável mais próxima, detecta falhas em segundos e drena o tráfego para longe, respeita a capacidade restante de cada região e absorve ataques DDoS volumétricos — tudo antes de a requisição chegar a um servidor de aplicação. O design gira em torno de dois mecanismos concorrentes — anycast e direcionamento por DNS — e suas características de failover muito diferentes. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Entendimento sólido de DNS, handshakes TCP/TLS e noções de BGP/anycast
- Familiaridade com conceitos de health check e circuit breaking
- Entendimento de medição de latência (RTT, geolocalização) e sua imprecisão
- Exposição a classes de ataque DDoS (volumétrico, de protocolo, de camada de aplicação)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Comparar anycast vs direcionamento por DNS e seus trade-offs de velocidade de failover
- Projetar um sistema de health check que evita tanto falsos positivos quanto flapping
- Rotear por latência e capacidade, não só geografia, e explicar a diferença
- Raciocinar sobre orçamentos de tempo de failover (TTL de DNS vs convergência BGP)
- Sobrepor mitigação de DDoS à frente da capacidade de origem

## Requisitos e Restrições

1. Rotear cada cliente para a região saudável de menor latência dentro de sua capacidade.
2. Detectar uma falha de região/PoP e drená-la em segundos, não minutos.
3. Sobreviver à perda de uma região inteira sem uma queda global.
4. Absorver DDoS volumétrico na borda antes que chegue à origem.
5. Evitar flaps de roteamento: uma região marginalmente insalubre não deve oscilar entrando e saindo.
6. Suportar direcionamento ponderado/canário para rollouts graduais.
7. Manter a própria decisão de direcionamento altamente disponível e de baixa latência.

## Abordagem Sugerida

Decida primeiro o mecanismo primário de direcionamento, pois ele limita sua velocidade de failover. **Anycast** (anunciar o mesmo IP de muitos PoPs; o BGP roteia para o mais próximo) faz failover em segundos, mas dá controle grosseiro e arrisca resets no meio da conexão em mudanças de rota. Direcionamento **por DNS** (o resolvedor retorna o IP da melhor região) dá controle fino e ciente de capacidade, mas é limitado pelo TTL de DNS e cache de resolvedor — muitas vezes dezenas de segundos a minutos. A maioria dos sistemas reais combina ambos: anycast até a borda, lógica DNS/borda para direcionar à origem. Projete health checks como sondas ativas mais sinais passivos, com histerese para prevenir flapping. Alimente métricas de capacidade na decisão para descartar carga antes de uma região saturar. Coloque a limpeza de DDoS na borda anycast, onde inundações volumétricas são absorvidas perto da fonte.

## Esboço de Arquitetura

```text
Cliente ──resolve DNS──> Svc GeoDNS/Direcionamento ──(saúde + capacidade + latência)──> IP da região
   │                          ^
   │                          └── Controlador de health check (sondas ativas + sinais passivos)
   ▼
PoP de borda anycast (limpeza DDoS, terminação TLS) ──> região saudável mais próxima ──> origem

Failover: região insalubre -> controlador marca DRAIN -> direcionamento para novo tráfego
          anycast: retirada BGP (segundos) | DNS: encurtar TTL, retornar alternativa (limitado por TTL)

APIs / plano de controle:
GET  /resolve?client_ip=...        -> { region, ip, ttl }
POST /health/report {region, rtt, errRate, capacityPct}
POST /steer/weight {region, weight} # canário / mudança gradual

Entradas de decisão:
Region{ id, healthy, capacityPct, measuredRttByGeo }  # histerese na flag healthy
```

## Tópicos de Aprofundamento

- **Anycast vs direcionamento por DNS:** trade-offs de velocidade de failover, granularidade, estabilidade de conexão.
- **Health checking:** ativo vs passivo, histerese/amortecimento de flap, detecção de falha parcial.
- **Roteamento ciente de capacidade:** descartar carga antes da saturação; evitar sobrecarga em cascata.
- **Orçamento de tempo de failover:** TTL de DNS e cache de resolvedor vs convergência BGP.
- **Mitigação de DDoS:** limpeza volumétrica na borda, dispersão anycast, rate limiting.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com o plano de controle de direcionamento e health check, refinado.
- [ ] Uma escolha fundamentada de anycast, DNS ou híbrido, com a análise de tempo de failover.
- [ ] O mecanismo de health check e amortecimento de flap especificado.
- [ ] Uma análise de falha/DR: perda de PoP único, perda de região inteira, queda do serviço de direcionamento.
- [ ] Uma seção de mitigação de DDoS cobrindo ataques volumétricos e de camada de aplicação.

## Armadilhas Comuns

- Assumir que o failover por DNS é instantâneo — caches de resolvedor ignoram seu TTL e servem IPs velhos por minutos.
- Health checks sem histerese, fazendo uma região marginal oscilar entrando e saindo da rotação.
- Rotear puramente por geografia, enviando tráfego à região mais próxima mesmo quando está sobrecarregada.
- Terminar tudo em um centro de limpeza, tornando-o o próprio alvo do DDoS.
- Esquecer que mudanças de rota anycast podem resetar conexões TCP em andamento.

## Recursos

- [Cloudflare: O que é Anycast?](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/) — direcionamento anycast e dispersão de DDoS explicados.
- [Artigo Maglev do Google](https://research.google/pubs/pub44824/) — um balanceador de carga de rede em software rápido e confiável.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de balanceamento de carga e disponibilidade.
- [AWS: políticas de roteamento Global Accelerator / Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html) — opções reais de roteamento geo/latência/ponderado.
- [RFC 1035: Nomes de Domínio](https://datatracker.ietf.org/doc/html/rfc1035) — DNS, TTLs e comportamento de resolução.
