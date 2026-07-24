# Sistema de Balanceamento de Carga

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um balanceador de carga que fica na frente de vários servidores backend e distribui o tráfego de entrada entre eles, o papel desempenhado por HAProxy, NGINX ou um balanceador L4/L7 na nuvem. Você vai implementar um ou mais algoritmos de distribuição, verificar ativamente a saúde de cada backend para que o tráfego nunca caia em um nó morto e drenar um backend graciosamente em vez de cortar suas requisições em andamento. O trabalho sutil está nas bordas: manter um cliente fixado a um backend quando as sessões exigem, decidir rápido o suficiente para que um nó falhando seja retirado antes de os usuários perceberem e degradar com bom senso quando todos os backends estão insalubres. Você sai entendendo por que um balanceador tem tanto a ver com saúde e failover quanto com "escolher o próximo servidor".

## Pré-requisitos

- Conforto para escrever um proxy HTTP ou TCP na linguagem de sua escolha
- Entender conexões, keep-alive e ciclos de vida de requisição/resposta
- Familiaridade com health checks (de um projeto de supervisor ou auto-escalonador)
- Noção básica de hashing para mapeamento consistente de cliente para backend
- Um trampolim: [Configuração de Auto-Escalonamento](../08-auto-scaling/) combina naturalmente com este

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar e comparar algoritmos: round-robin, least-connections e variantes ponderadas
- Verificar ativamente a saúde dos backends e removê-los/restaurá-los da rotação automaticamente
- Suportar sticky sessions via cookie ou hashing consistente quando necessário
- Drenar conexões na remoção para que requisições em andamento concluam antes de um backend sair
- Degradar graciosamente — retornar um erro claro, não um travamento, quando nenhum backend está disponível

## Requisitos Funcionais

1. O balanceador deve aceitar requisições de clientes e encaminhá-las a um backend saudável.
2. Deve suportar ao menos seleção round-robin e least-connections, escolhida por configuração.
3. Deve verificar ativamente a saúde de cada backend e excluir os que falham em tempo limitado.
4. Um backend recuperado deve reingressar na rotação automaticamente.
5. Sticky sessions devem rotear um dado cliente ao mesmo backend enquanto ele permanecer saudável.
6. Remover um backend deve drenar as conexões existentes em vez de derrubá-las.
7. Quando todos os backends estão insalubres, o balanceador deve retornar um erro definido, não travar nem quebrar.

## Marcos Sugeridos

1. **Marco 1 — Encaminhar e rotacionar:** Faça proxy de requisições a um pool estático usando round-robin.
2. **Marco 2 — Saúde e failover:** Adicione health checks ativos, exclusão e recuperação automática.
3. **Marco 3 — Fixação e drenagem:** Adicione afinidade de sessão e drenagem graciosa de conexões.

## Esboço de Dados e Interface

```text
backend
  id          string
  address     host:port
  weight      int
  status      up | down | draining
  in_flight   int          (para least-connections)

config
  algorithm     round_robin | least_conn | weighted
  health        { path, interval_s, timeout_s, unhealthy_after, healthy_after }
  sticky        none | cookie | ip_hash

seleção:
  round_robin   próximo índice mod pool
  least_conn    backend com menor in_flight entre os up
  weighted      distribuir proporcional ao weight

caminho da requisição:
  client -> balancer -> escolher backend (respeitar sticky) -> proxy
         backend cai no meio da requisição -> retry em outro (só idempotente)
         nenhum backend up -> 503 com corpo claro

laço de saúde marca up/down após N resultados consecutivos
```

## Desafios Extras

- Adicione pooling de conexões e reuso de keep-alive para os backends upstream.
- Adicione rate limiting por backend e um circuit breaker simples para nós que falham repetidamente.
- Exponha um endpoint de métricas (requisições, latência, saúde por backend) e uma pequena visão de admin.
- Suporte roteamento canário ponderado para enviar uma pequena porcentagem do tráfego a um novo backend.

## Definição de Pronto

- [ ] O tráfego é distribuído entre os backends conforme o algoritmo configurado.
- [ ] Um backend que falha seu health check é removido dentro da janela configurada e reingressa depois na recuperação.
- [ ] Sticky sessions mantêm um cliente em um backend durante toda a sessão enquanto ele está saudável.
- [ ] Remover um backend drena as requisições em andamento em vez de rompê-las.
- [ ] Com todos os backends fora, os clientes recebem uma resposta de erro definida, não um timeout.

## Armadilhas Comuns

- Health checks que só verificam o connect TCP, deixando passar um backend que aceita conexões mas retorna 500s.
- Fazer retry de requisições não idempotentes no failover, causando efeitos colaterais duplicados.
- Sticky sessions sem fallback, deixando um cliente encalhado quando o backend fixado morre.
- Remover um backend instantaneamente a uma única checagem falha, causando flapping sob instabilidades transitórias.
- Cortar conexões na drenagem, quebrando requisições longas e uploads.

## Recursos

- [Documentação do HAProxy](https://docs.haproxy.org/) — algoritmos, health checks e stick tables em um balanceador real.
- [NGINX: HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) — configuração prática dos conceitos aqui.
- [Cloudflare: What is Load Balancing?](https://www.cloudflare.com/learning/performance/what-is-load-balancing/) — uma visão conceitual clara.
- [Google SRE Book: Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/) — balanceamento consciente de saúde em escala.
