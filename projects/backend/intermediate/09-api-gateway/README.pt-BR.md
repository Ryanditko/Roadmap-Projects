# API Gateway (roteamento básico)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa a porta de entrada única que fica na frente de vários serviços de backend e decide para onde cada requisição vai. Os clientes falam apenas com o gateway; ele inspeciona o caminho, escolhe o upstream certo, encaminha a requisição e devolve a resposta. No processo, ele faz o trabalho transversal que nenhum serviço individual deveria repetir — validar o JWT uma vez, aplicar timeouts, repetir falhas transitórias e abrir um circuit breaker quando um upstream adoece. É assim que ferramentas como Kong, NGINX e AWS API Gateway ganham seu espaço, e construir um pequeno ensina onde latência, falha e fronteiras de confiança realmente moram em um sistema distribuído.

## Pré-requisitos

- Conforto para escrever servidores HTTP e fazer chamadas HTTP de saída na linguagem de sua escolha
- Entendimento de verificação de JWT (um aquecimento é a [API de E-commerce com JWT](../01-ecommerce-api-jwt/))
- Familiaridade com códigos de status HTTP, cabeçalhos e o ciclo requisição/resposta
- Dois ou três serviços de backend minúsculos (ou stubs) para rotear
- Noção básica de concorrência e reuso de conexões

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Rotear requisições para o upstream correto usando regras baseadas em caminho
- Validar autenticação de forma centralizada para que os upstreams confiem no gateway
- Distribuir carga entre réplicas saudáveis com balanceamento round-robin
- Aplicar timeouts e retries limitados por upstream sem amplificar a falha
- Implementar um circuit breaker que isola um serviço em falha e se recupera automaticamente
- Distinguir erros originados no gateway (502/503/504) dos erros do upstream

## Requisitos Funcionais

1. O gateway deve encaminhar requisições para um backend escolhido comparando o caminho da requisição com um registro de serviços.
2. Deve retornar 404 quando nenhuma rota casar e 502/503 quando o upstream escolhido estiver inacessível.
3. Deve validar o token bearer do `Authorization` uma vez e rejeitar tokens inválidos com 401 antes de encaminhar.
4. Deve balancear carga entre múltiplas instâncias do mesmo serviço usando round-robin.
5. Deve aplicar um timeout configurável por requisição e retornar 504 quando um upstream excedê-lo.
6. Deve repetir requisições idempotentes um número limitado de vezes em falhas transitórias (5xx/conexão).
7. Deve abrir um circuit breaker após falhas repetidas do upstream e falhar rápido até uma sondagem half-open ter sucesso.
8. Deve encaminhar o método, cabeçalhos, query string e corpo originais, preservando o status e o corpo do upstream na volta.

## Marcos Sugeridos

1. **Marco 1 — Proxy reverso:** Case um caminho a um upstream e encaminhe fielmente requisição e resposta.
2. **Marco 2 — Registro e balanceamento:** Suporte múltiplos serviços com múltiplas instâncias e seleção round-robin.
3. **Marco 3 — Auth e resiliência:** Adicione validação central de JWT, timeouts, retries limitados e um circuit breaker com health checks.

## Esboço de Dados e Interface

```text
Registro de serviços (config)
  services:
    - name: "orders"
      prefix: "/orders"
      instances: ["http://orders-1:8080", "http://orders-2:8080"]
      timeoutMs: 2000
      auth: true

Fluxo da requisição
  cliente -> gateway
    casar prefixo -> serviço
    se serviço.auth: verificar JWT (401 em falha)
    escolher instância (round-robin, pular circuitos abertos)
    encaminhar com timeout -> upstream
    em 5xx/timeout: retry (só idempotente, máx N)
    abrir breaker após K falhas consecutivas

Estados do breaker: FECHADO -> ABERTO -> MEIO-ABERTO -> FECHADO
Erros do gateway: 404 sem rota | 401 token inválido | 502 upstream ruim
                  503 circuito aberto | 504 timeout
```

## Desafios Extras

- Adicione agregação de resposta: uma rota do gateway que dispara para dois upstreams e mescla os resultados.
- Suporte transformação de cabeçalhos de requisição/resposta (injetar `X-Request-Id`, remover cabeçalhos hop-by-hop).
- Adicione balanceamento ponderado para que uma instância canário receba uma pequena porcentagem do tráfego.
- Exponha um endpoint de métricas reportando latência, taxa de erro e estado do breaker por upstream.

## Definição de Pronto

- [ ] Uma requisição a um prefixo conhecido chega ao upstream certo e a resposta é devolvida inalterada.
- [ ] Caminhos desconhecidos retornam 404 e upstreams inacessíveis retornam 502/503, nunca um travamento.
- [ ] Um token inválido ou ausente é rejeitado com 401 antes de qualquer chamada ao upstream.
- [ ] O tráfego se espalha uniformemente entre instâncias saudáveis, e uma instância derrubada é ignorada.
- [ ] Um upstream lento dispara o timeout (504); falhas repetidas abrem o breaker e ele se recupera depois.

## Armadilhas Comuns

- Encaminhar cabeçalhos hop-by-hop (`Connection`, `Transfer-Encoding`) literalmente, corrompendo a resposta encaminhada.
- Repetir requisições não idempotentes (POST de pagamentos), causando efeitos colaterais duplicados.
- Nenhum timeout na chamada de saída, então um upstream lento esgota o pool de conexões do gateway.
- Tratar um 401 do upstream como erro do gateway, escondendo a causa real do cliente.
- Um circuit breaker que abre mas nunca sonda, bloqueando permanentemente um serviço recuperado.

## Recursos

- [NGINX: O que é um API Gateway?](https://www.nginx.com/learn/api-gateway/) — o padrão e suas responsabilidades.
- [Martin Fowler: CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html) — a explicação canônica dos estados e transições.
- [MDN: Servidores proxy e tunelamento](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Proxy_servers_and_tunneling) — cabeçalhos hop-by-hop vs end-to-end.
- [roadmap.sh: Desenvolvedor Backend](https://roadmap.sh/backend) — onde gateways se encaixam no cenário mais amplo de backend.
