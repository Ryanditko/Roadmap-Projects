# Sistema de Verificação de Saúde

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um pequeno sistema que repetidamente pergunta aos seus serviços "você está bem?" e faz algo útil quando a resposta é não. Ele sonda cada alvo em um cronograma — um endpoint HTTP, uma porta TCP, uma dependência como um banco de dados —, registra o resultado e alerta quando um alvo permanece fora do ar além de uma tolerância que você define. A sutileza está em distinguir uma queda real de uma oscilação: uma sonda falha não deve acordar ninguém, mas três seguidas devem. Você também aprenderá a diferença entre liveness ("está rodando?") e readiness ("consegue servir tráfego?"), uma distinção que sustenta o modelo de saúde de todo orquestrador.

## Pré-requisitos

- Um ou mais serviços com um endpoint ou porta acessível
- Uma linguagem de script que consiga fazer requisições HTTP/TCP
- Entendimento de códigos de status HTTP e timeouts
- Uma forma de notificar (console, log ou webhook de chat)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Sondar alvos HTTP e TCP em um intervalo fixo com timeouts
- Distinguir liveness de readiness e checar dependências
- Exigir falhas consecutivas antes de declarar um alvo fora do ar
- Alertar em uma mudança de estado (up→down, down→up), não a cada sonda
- Registrar o histórico de sondas para calcular um uptime simples

## Requisitos Funcionais

1. O sistema deve checar uma lista configurável de alvos em um intervalo fixo.
2. Cada sonda deve impor um timeout e tratar um timeout como falha.
3. Um alvo deve ser marcado como fora do ar apenas após N falhas consecutivas (configurável).
4. Ele deve alertar uma vez em cada transição de estado, não a cada sonda falha.
5. Ele deve suportar ao menos HTTP (status + corpo/palavra-chave) e checagens de porta TCP.
6. Ele deve registrar cada resultado para que o uptime em um período possa ser reportado.
7. Alvos, intervalos, limiares e timeouts devem ser configuráveis.

## Marcos Sugeridos

1. **Marco 1 — Sondar e reportar:** Cheque uma lista de alvos HTTP em um intervalo e imprima up/down.
2. **Marco 2 — Debounce e alerta:** Exija N falhas consecutivas e alerte apenas em transições de estado.
3. **Marco 3 — Histórico e uptime:** Persista resultados e exponha um resumo de uptime e uma visão de status simples.

## Esboço de Dados e Interface

```text
Config (estrutura, não o arquivo completo)
  interval_seconds: 30
  targets:
    - name: api
      type: http
      url: http://localhost:8080/health
      expect_status: 200
      timeout_ms: 2000
      unhealthy_after: 3     # falhas consecutivas
    - name: db
      type: tcp
      host: localhost
      port: 5432
      timeout_ms: 1000

Máquina de estados por alvo
  UP --(N falhas consecutivas)--> DOWN   (alerta)
  DOWN --(1 sucesso)--> UP                (alerta de recuperação)

Registro de resultado
  { target, ok, latency_ms, checked_at, consecutive_failures }
```

## Desafios Extras

- Adicione uma distinção readiness vs liveness e cheque dependências downstream separadamente.
- Sirva uma pequena página de status listando o estado atual de cada alvo e o uptime de 24h.
- Adicione jitter ao tempo das sondas para que nem todas disparem no mesmo instante.
- Escale: avise após N falhas, acione um page após M.

## Definição de Pronto

- [ ] Alvos são sondados no intervalo configurado com timeouts impostos.
- [ ] Uma única sonda falha não dispara um alerta; N seguidas disparam.
- [ ] A recuperação (down→up) produz um alerta distinto.
- [ ] O uptime em uma janela pode ser reportado a partir do histórico registrado.
- [ ] Todos os limiares e alvos vêm da config, não do código.

## Armadilhas Comuns

- Sem timeout nas sondas, um alvo travado paralisa todo o loop de checagem.
- Alertar a cada sonda falha em vez de na mudança de estado, causando fadiga de alertas.
- Tratar qualquer 2xx–3xx como saudável quando a app retorna 200 com um corpo de erro.
- Checar apenas liveness, de modo que um processo que está de pé mas não alcança seu banco pareça saudável.

## Recursos

- [Kubernetes: Probes de liveness, readiness e startup](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — o modelo canônico.
- [Microsoft: Padrão Health Endpoint Monitoring](https://learn.microsoft.com/en-us/azure/architecture/patterns/health-endpoint-monitoring) — projetando endpoints de saúde.
- [MDN: Códigos de status de resposta HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status) — o que "saudável" realmente significa.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — sinais, sintomas e alertas.
