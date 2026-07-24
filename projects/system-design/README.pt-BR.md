![System Design Projects Banner](../images/banner-system-design.png)

> 🌐 [English](./README.md) · **Português**

# Projetos de System Design

Projete sistemas distribuídos de larga escala: escalabilidade, confiabilidade e
os trade-offs de arquitetura por trás de produtos reais. São exercícios
guiados — você recebe o *o quê* e o *porquê*, e então produz o design e o raciocínio.

> Cada projeto tem um brief em inglês (`README.md`) e outro em português
> (`README.pt-BR.md`).

## O Que Você Vai Construir

- Designs de componentes para encurtadores de URL, chat e rate limiters
- Arquiteturas para e-commerce, streaming e feeds sociais
- Blueprints para sistemas de escala planetária (Netflix, Uber, WhatsApp)
- Bancos de dados distribuídos e balanceadores de carga globais
- Documentos de trade-off fundamentados, não apenas diagramas

## O Que Você Vai Aprender

- **Escalabilidade**: escalonamento horizontal, sharding, particionamento
- **Consistência**: teorema CAP, ACID vs. BASE, replicação
- **Cache**: estratégias, invalidação, caches distribuídos
- **Comunicação**: síncrona vs. assíncrona, filas, pub/sub
- **Trade-offs**: raciocínio sobre falhas, latência e custo

## Projetos Iniciantes (10 Projetos)

Conceitos fundamentais de system design por meio de problemas focados.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [Design URL Shortener](./beginner/01-url-shortener/) | Crie um sistema que encurta URLs longas |
| 2 | [Design Chat System](./beginner/02-chat-system/) | Arquitete uma plataforma de mensagens para usuários |
| 3 | [Design File Storage System](./beginner/03-file-storage/) | Construa infraestrutura para armazenar e servir arquivos |
| 4 | [Design Notification System](./beginner/04-notification-system/) | Crie um sistema que envia notificações aos usuários |
| 5 | [Design Login System](./beginner/05-login-system/) | Arquitete autenticação de usuários segura |
| 6 | [Design Rate Limiter](./beginner/06-rate-limiter/) | Previna abusos limitando a taxa de requisições |
| 7 | [Design Cache System](./beginner/07-cache-system/) | Construa uma camada de cache eficiente |
| 8 | [Design Task Scheduler](./beginner/08-task-scheduler/) | Crie um sistema para agendar e executar tarefas |
| 9 | [Design Logging System](./beginner/09-logging-system/) | Construa infraestrutura para logging de aplicações |
| 10 | [Design Metrics System](./beginner/10-metrics-system/) | Crie sistemas para coletar e agregar métricas |

## Projetos Intermediários (10 Projetos)

Vários conceitos combinados em padrões reais de design.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [Design Scalable E-Commerce System](./intermediate/01-scalable-ecommerce/) | Arquitetura para plataforma de compras online |
| 2 | [Design Video Streaming System](./intermediate/02-video-streaming/) | Infraestrutura para entrega de vídeo em escala |
| 3 | [Design Ride-Sharing System](./intermediate/03-ride-sharing/) | Design de sistema para plataforma estilo Uber |
| 4 | [Design Social Media Feed](./intermediate/04-social-feed/) | Arquitetura para feeds de conteúdo personalizados |
| 5 | [Design Search Engine](./intermediate/05-search-engine/) | Sistema para busca full-text e ranqueamento |
| 6 | [Design Messaging Queue](./intermediate/06-messaging-queue/) | Construa processamento assíncrono de mensagens |
| 7 | [Design API Gateway](./intermediate/07-api-gateway/) | Crie uma camada de roteamento e gestão |
| 8 | [Design Recommendation System](./intermediate/08-recommendation-system/) | Sistema que sugere conteúdo relevante aos usuários |
| 9 | [Design Analytics System](./intermediate/09-analytics-system/) | Infraestrutura para coleta e análise de dados |
| 10 | [Design CDN](./intermediate/10-cdn/) | Rede de distribuição de conteúdo para alcance global |

## Projetos Avançados (10 Projetos)

Arquitete sistemas de escala global com preocupações de nível empresarial.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [Design Netflix-Like System](./advanced/01-netflix-system/) | Arquitetura para uma plataforma global de streaming de vídeo |
| 2 | [Design Uber-Like System](./advanced/02-uber-system/) | Matching e despacho em tempo real em escala massiva |
| 3 | [Design WhatsApp-Like System](./advanced/03-whatsapp-system/) | Plataforma global de mensagens com bilhões de usuários |
| 4 | [Design YouTube System](./advanced/04-youtube-system/) | Plataforma de vídeo suportando bilhões de visualizações |
| 5 | [Design Amazon System](./advanced/05-amazon-system/) | E-commerce em escala global com logística |
| 6 | [Design Distributed Database](./advanced/06-distributed-database/) | Sistema de banco de dados abrangendo múltiplas regiões |
| 7 | [Design Global Load Balancer](./advanced/07-global-load-balancer/) | Roteamento geográfico e gestão de tráfego |
| 8 | [Design Real-Time Collaboration System](./advanced/08-collaboration-system/) | Edição concorrente estilo Google Docs |
| 9 | [Design High-Frequency Trading System](./advanced/09-hft-system/) | Infraestrutura de trading de latência ultrabaixa |
| 10 | [Design Multi-Region Architecture](./advanced/10-multi-region-architecture/) | Design de sistema para tolerância a falhas global |

## Trilha de Aprendizado

- **Iniciante (3–4 semanas)**: vocabulário base, limites de servidor único,
  componentes fundamentais (bancos de dados, cache), trade-offs básicos.
- **Intermediário (6–8 semanas)**: sistemas de média escala, microsserviços,
  modelos de consistência, arquiteturas do mundo real.
- **Avançado (2–3 meses)**: design em escala planetária, trade-offs complexos,
  problemas inéditos sem templates.

Comece simples e depois itere — sempre declare suas premissas e trade-offs.

## Conceitos-Chave

1. **Escalonamento** — horizontal vs. vertical, auto-scaling
2. **Bancos de dados** — SQL vs. NoSQL, replicação, sharding
3. **Cache** — padrões e estratégias de invalidação
4. **Balanceamento de carga** — algoritmos, sticky vs. stateless
5. **Consistência** — teorema CAP, consistência eventual
6. **Mensageria** — garantias de entrega, ordenação, DLQs
7. **Resiliência** — pontos únicos de falha, failover

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Designing Data-Intensive Applications](https://dataintensive.io/)
- [High Scalability Blog](http://highscalability.com/)
- [Papers We Love](https://paperswelove.org/)
- [Roadmap de Design e Arquitetura de Software](https://roadmap.sh/software-design-architecture)

## Próximos Passos

1. Leia os fundamentos de "Designing Data-Intensive Applications".
2. Escolha um projeto iniciante e leia o brief até o fim.
3. Desenhe diagramas de arquitetura e documente seus trade-offs.
4. Apresente seu design para receber feedback e depois tente os desafios extras.

**Escolha um projeto e comece a arquitetar em escala.**
