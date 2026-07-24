# Monitor de Reinício de Serviços

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um pequeno supervisor que observa um conjunto de serviços locais, percebe quando um deles morre ou para de responder e o reinicia automaticamente. Essa é a ideia por trás de ferramentas como systemd, supervisord e os liveness probes do Kubernetes, reduzida à sua essência: um laço que verifica a saúde, decide se um processo está vivo e age. A parte interessante não é o reinício em si, mas a disciplina ao seu redor — saber a diferença entre "não está rodando" e "está rodando mas não saudável", recuar para não martelar um serviço quebrado e parar após falhas demais em vez de reiniciar para sempre. Você terminará com um modelo mental claro do que um supervisor de processos realmente faz e por que scripts ingênuos de "só reinicia" causam mais quedas do que resolvem.

## Pré-requisitos

- Conforto para rodar e parar processos a partir de um shell
- Scripting básico em qualquer linguagem (Bash, Python, Go ou Node)
- Entender códigos de saída e como um processo reporta sucesso ou falha
- Familiaridade com leitura de logs e timestamps

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Distinguir liveness (o processo está rodando?) de readiness (ele está de fato servindo?)
- Implementar uma verificação de saúde via status do processo, uma porta TCP ou um endpoint HTTP
- Aplicar backoff exponencial para que as tentativas desacelerem em vez de virar um laço apertado
- Impor um limite de reinícios e entrar em um estado de "desistência" que alerta em vez de tentar de novo
- Manter um histórico auditável de falhas e decisões de reinício

## Requisitos Funcionais

1. O monitor deve acompanhar uma lista configurável de serviços, cada um com sua própria verificação de saúde.
2. Ele deve detectar quando um serviço está fora (processo morto, porta fechada ou endpoint falhando).
3. Ao falhar, deve tentar um reinício e registrar que o fez.
4. As novas tentativas devem usar backoff exponencial com base e teto configuráveis.
5. Após um número configurável de falhas em uma janela, deve parar de reiniciar e emitir um alerta.
6. Uma sobreposição manual deve permitir a um operador desabilitar ou forçar o reinício de um serviço específico.
7. Toda mudança de estado (fora, reiniciando, recuperado, desistiu) deve ser registrada com timestamp.

## Marcos Sugeridos

1. **Marco 1 — Detectar e reiniciar:** Monitore um serviço, detecte um processo morto, reinicie-o, registre o evento.
2. **Marco 2 — Backoff e limites:** Adicione backoff exponencial e um limite máximo de reinícios com estado de desistência.
3. **Marco 3 — Múltiplos serviços e sobreposição:** Controle vários serviços via configuração e adicione controles manuais.

## Esboço de Dados e Interface

```text
config (por serviço)
  name           string
  start_cmd      string
  check          { type: process | tcp | http, target, interval_s }
  backoff        { base_s, max_s }
  max_restarts   int   (dentro de window_s)

estado do serviço (em memória)
  status         running | down | restarting | gave_up
  failures       int
  next_attempt   timestamp
  last_change    timestamp

superfície de controle (CLI ou arquivo)
  status                 -> tabela de serviços + estado
  disable <name>         -> parar de monitorar
  restart <name>         -> forçar um reinício

decisão de reinício:
  down -> tentar se failures < max_restarts
       -> aguardar min(base * 2^failures, max) antes da próxima tentativa
       -> failures >= max na janela -> gave_up + alerta
```

## Desafios Extras

- Adicione dependências entre serviços para que um reinício cascateie na ordem correta.
- Envie notificações a um webhook de chat (ex.: Slack) quando um serviço entrar em desistência.
- Exponha um pequeno painel de status ou um endpoint `/healthz` para o próprio monitor.
- Persista o histórico em arquivo para que as contagens de reinício sobrevivam a um reinício do monitor.

## Definição de Pronto

- [ ] Um serviço encerrado é detectado e reiniciado em até um intervalo de verificação.
- [ ] Falhas repetidas recuam exponencialmente em vez de tentar de novo imediatamente.
- [ ] Após atingir o limite de reinícios, o serviço entra em desistência e alerta em vez de entrar em laço.
- [ ] Um serviço saudável novamente zera seu contador de falhas e volta ao monitoramento normal.
- [ ] Toda transição é registrada com timestamp e permanece legível depois.

## Armadilhas Comuns

- Tratar "o processo existe" como "saudável" — um processo travado passa na checagem de PID mas não serve nada.
- Reiniciar sem backoff, transformando um crash loop em um laço quente que trava a CPU.
- Nunca desistir, de modo que um serviço permanentemente quebrado esconde o problema real por trás de reinícios infinitos.
- Esquecer de zerar o contador de falhas após a recuperação, causando desistência prematura mais tarde.
- Corrida no reinício: subir uma segunda cópia antes de confirmar que a primeira realmente se foi.

## Recursos

- [Manual do systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) — como um supervisor real modela política de reinício.
- [Documentação do Supervisor](http://supervisord.org/) — um sistema de controle de processos com exatamente os conceitos aqui.
- [Kubernetes: Configurar Liveness, Readiness e Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — liveness vs readiness na prática.
- [Google SRE Book: Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/) — por que backoff e limites importam.
