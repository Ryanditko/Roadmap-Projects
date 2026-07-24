![Backend Projects Banner](../images/banner-backend.png)

> 🌐 [English](./README.md) · **Português**

# Projetos de Desenvolvimento Backend

Construa o lado servidor de aplicações reais: APIs, autenticação,
persistência de dados, filas de mensagens e serviços distribuídos. São
exercícios guiados — você recebe o *o quê* e o *porquê*, e então escreve o código.

> Cada projeto tem um brief em inglês (`README.md`) e outro em português
> (`README.pt-BR.md`).

## O Que Você Vai Construir

- APIs REST e GraphQL com contratos limpos e versionados
- Fluxos de autenticação e autorização (sessões, JWT, OAuth2)
- Processadores de tarefas em background e serviços orientados a eventos
- Camadas de cache, limitadores de taxa e integrações resilientes
- Sistemas distribuídos com observabilidade e tolerância a falhas

## O Que Você Vai Aprender

- **Design de APIs**: princípios REST, GraphQL, gRPC
- **Gestão de dados**: modelagem SQL/NoSQL, índices, transações
- **Segurança**: hash de senhas, fluxos de token, isolamento de tenants
- **Escalabilidade**: cache, filas, balanceamento de carga, sharding
- **Confiabilidade**: retentativas, idempotência, circuit breakers, monitoramento

## Projetos Iniciantes (10 Projetos)

Conceitos fundamentais de backend por meio de serviços pequenos e focados.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [Simple REST API for Task Management](./beginner/01-simple-rest-api-task-management/) | Construa uma API REST básica para gerenciar tarefas com operações CRUD |
| 2 | [URL Shortener (In-Memory)](./beginner/02-url-shortener-in-memory/) | Crie um serviço que encurta URLs e armazena os mapeamentos em memória |
| 3 | [Basic Authentication API](./beginner/03-basic-authentication-api/) | Implemente login simples com credenciais fixas |
| 4 | [Notes API with File Persistence](./beginner/04-notes-api-file-persistence/) | Construa uma API que salva notas no sistema de arquivos |
| 5 | [Weather Proxy API](./beginner/05-weather-proxy-api/) | Crie um wrapper de API sobre um serviço externo de clima |
| 6 | [CLI User Manager](./beginner/06-cli-user-manager/) | Construa uma ferramenta de linha de comando para gerenciar dados de usuários |
| 7 | [File Upload API](./beginner/07-file-upload-api/) | Implemente upload de arquivos com armazenamento local |
| 8 | [Basic Logging System](./beginner/08-basic-logging-system/) | Crie um serviço simples de logging para eventos da aplicação |
| 9 | [Static JSON API Server](./beginner/09-static-json-api-server/) | Construa um servidor de API mínimo servindo JSON estático |
| 10 | [Email Sender Mock Service](./beginner/10-email-sender-mock-service/) | Crie um serviço que simula o envio de e-mails |

## Projetos Intermediários (10 Projetos)

Vários conceitos combinados em padrões reais de backend.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [E-commerce API with JWT](./intermediate/01-ecommerce-api-jwt/) | Construa uma API de catálogo de produtos com autenticação JWT |
| 2 | [Blog Platform with CRUD + Comments](./intermediate/02-blog-platform-crud/) | Crie um motor de blog com posts, comentários e gestão de usuários |
| 3 | [Rate-Limited API with Redis](./intermediate/03-rate-limited-api-redis/) | Implemente limitação de taxa de API usando cache Redis |
| 4 | [Job Queue System](./intermediate/04-job-queue-rabbitmq/) | Construa um processador de tarefas em background com filas |
| 5 | [Multi-Tenant SaaS API](./intermediate/05-multi-tenant-saas-api/) | Projete uma API servindo múltiplos tenants isolados |
| 6 | [API with Caching Layer](./intermediate/06-api-caching-redis/) | Adicione cache Redis para melhorar o desempenho da API |
| 7 | [Payment Processing Mock Service](./intermediate/07-payment-mock-service/) | Crie um gateway de pagamento simulado com tratamento de transações |
| 8 | [Notification Service](./intermediate/08-notification-service/) | Construa um serviço que envia e-mails e SMS com lógica de retentativa |
| 9 | [API Gateway](./intermediate/09-api-gateway/) | Implemente um gateway para roteamento e gestão de APIs |
| 10 | [GraphQL API with Resolvers](./intermediate/10-graphql-api/) | Construa um servidor GraphQL com busca de dados eficiente |

## Projetos Avançados (10 Projetos)

Arquitete sistemas complexos com preocupações de nível empresarial.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [Distributed Order Processing System](./advanced/01-distributed-order-system/) | Projete um sistema de pedidos de alta disponibilidade com múltiplos serviços |
| 2 | [Event-Driven Microservices Architecture](./advanced/02-event-driven-microservices/) | Construa um sistema de serviços comunicando-se por eventos |
| 3 | [High-Scale Authentication Service](./advanced/03-oauth2-auth-service/) | Crie um serviço de autenticação para milhões de usuários |
| 4 | [Real-Time Chat Backend](./advanced/04-realtime-chat-websockets/) | Construa um servidor de chat com conexões WebSocket |
| 5 | [Feature Flag Service](./advanced/05-feature-flag-service/) | Implemente um sistema para gerenciar feature toggles |
| 6 | [Observability Platform](./advanced/06-observability-platform/) | Construa uma infraestrutura de logs, métricas e tracing |
| 7 | [Resilient API with Circuit Breaker](./advanced/07-resilient-api-circuit-breaker/) | Projete APIs tolerantes a falhas com padrões de circuit breaker |
| 8 | [Multi-Region API Design](./advanced/08-multi-region-api/) | Arquitete APIs implantadas em múltiplas regiões geográficas |
| 9 | [Streaming Platform Backend](./advanced/09-streaming-platform-backend/) | Construa o backend de um serviço de vídeo estilo Netflix |
| 10 | [Idempotent Financial Transactions](./advanced/10-idempotent-financial-transactions/) | Projete sistemas que lidam com operações financeiras com segurança |

## Trilha de Aprendizado

- **Iniciante (2–4 semanas)**: fundamentos de HTTP e REST, roteamento e
  middleware, armazenamento em memória, autenticação simples.
- **Intermediário (4–8 semanas)**: bancos de dados reais, JWT/OAuth, camadas
  de cache, versionamento e design de API.
- **Avançado (2–3 meses)**: sistemas distribuídos, microsserviços,
  escalabilidade, observabilidade e trade-offs de confiabilidade.

Complete os projetos de um nível antes de subir — cada nível assume o anterior.

## Conceitos-Chave

1. **Design de API** — contratos REST, GraphQL, gRPC
2. **Bancos de dados** — modelagem de schema, índices, otimização de consultas
3. **Autenticação** — sessões, JWT, OAuth2
4. **Cache** — estratégias de invalidação, read-through/write-through
5. **Mensageria** — tarefas em background, pub/sub, streaming de eventos
6. **Resiliência** — retentativas, idempotência, circuit breakers
7. **Testes** — unitários, integração e de contrato

## Recursos

- [REST API Best Practices](https://restfulapi.net/)
- [Documentação do PostgreSQL](https://www.postgresql.org/docs/)
- [OpenAPI / Swagger](https://swagger.io/)
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Roadmap de Backend](https://roadmap.sh/backend)

## Próximos Passos

1. Escolha um projeto iniciante e leia o brief até o fim.
2. Configure seu ambiente na stack que preferir.
3. Construa marco a marco.
4. Verifique a Definição de Pronto e depois tente os desafios extras.

**Escolha um projeto e comece a construir.**
