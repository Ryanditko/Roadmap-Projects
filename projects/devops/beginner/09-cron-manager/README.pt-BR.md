# Gerenciador de Cron Jobs

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

O `cron` puro roda seus jobs mas quase não conta nada: sem histórico, sem alerta quando um job falha, sem proteção contra uma execução lenta que se sobrepõe à próxima. Construa um gerenciador enxuto que envolve jobs agendados e adiciona a camada operacional que lhes falta. Ele lê uma lista de jobs e seus cronogramas, roda cada um na hora certa, captura a saída e o código de saída, impede que um job rode enquanto uma instância anterior ainda está em andamento e alerta quando um falha ou estoura o tempo. Ao terminar, você entenderá o que o cron de fato garante — e o que você precisa adicionar por conta própria para confiar nele em produção.

## Pré-requisitos

- Alguns comandos ou scripts que você quer rodar em um cronograma
- Uma linguagem de script para o gerenciador (Python, Go ou shell)
- Entendimento de processos, códigos de saída e saída padrão/de erro
- Familiaridade com a sintaxe de expressões cron

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Interpretar cronogramas no estilo cron e determinar quando cada job está devido
- Rodar um job como subprocesso e capturar sua saída e código de saída
- Impedir execuções sobrepostas do mesmo job com um lock
- Registrar um histórico de execuções e alertar em falha ou estouro de tempo
- Impor um timeout por job para que um job travado não rode para sempre

## Requisitos Funcionais

1. O gerenciador deve ler uma lista de jobs, cada um com um comando e um cronograma cron.
2. Ele deve rodar cada job no horário agendado e capturar stdout, stderr e código de saída.
3. Ele deve impedir que um job inicie se sua execução anterior ainda estiver em andamento (locking).
4. Ele deve registrar uma entrada de histórico por execução: início, fim, código de saída e status.
5. Ele deve alertar quando um job sai com código diferente de zero ou excede seu timeout configurado.
6. Ele deve terminar um job que excede seu timeout.
7. Jobs, cronogramas, timeouts e configurações de alerta devem ser configuráveis.

## Marcos Sugeridos

1. **Marco 1 — Agendar e rodar:** Interprete cronogramas, rode jobs na hora e capture saída e código de saída.
2. **Marco 2 — Lock e histórico:** Adicione locking por job contra sobreposição e persista um histórico de execuções.
3. **Marco 3 — Timeout e alerta:** Imponha timeouts, mate estouros e alerte em falha ou estouro de tempo.

## Esboço de Dados e Interface

```text
Config de job (estrutura, não o arquivo completo)
  jobs:
    - name: nightly-report
      schedule: "0 3 * * *"     # expressão cron
      command: ["/usr/bin/report", "--daily"]
      timeout_seconds: 600
      on_failure: alert
    - name: cache-warm
      schedule: "*/15 * * * *"
      command: ["./warm-cache.sh"]
      allow_overlap: false

Registro de execução (persistido por execução)
  { job, run_id, started_at, ended_at, exit_code, status, output_ref }
  status: success | failed | timed_out | skipped_locked

CLI
  manager run              # daemon: avalia cronogramas, dispara jobs devidos
  manager list             # jobs + próximo horário de execução
  manager history <job>    # execuções recentes e resultados
```

## Desafios Extras

- Adicione dependências entre jobs: rode B só depois que A tiver sucesso.
- Adicione uma política de retry com backoff para falhas transitórias.
- Exponha uma pequena página ou endpoint de status com os resultados das últimas execuções.
- Suporte variáveis de ambiente e um diretório de trabalho por job.

## Definição de Pronto

- [ ] Jobs rodam nos horários agendados e seus códigos de saída são registrados.
- [ ] Um job de longa duração bloqueia sua própria próxima execução em vez de se sobrepor.
- [ ] Um job que excede seu timeout é morto e marcado como `timed_out`.
- [ ] Uma saída diferente de zero dispara exatamente um alerta com o nome do job e a saída.
- [ ] O histórico mostra resultados por execução e é consultável por job.

## Armadilhas Comuns

- Assumir que a execução anterior terminou; sem um lock, um job lento acumula cópias de si mesmo.
- Descartar stdout/stderr, de modo que um job com falha não deixa pista do porquê.
- Ignorar o código de saída e tratar "rodou" como "teve sucesso".
- Agendar em horário local e se surpreender com mudanças de horário de verão — prefira UTC.

## Recursos

- [Manual do crontab(5)](https://man7.org/linux/man-pages/man5/crontab.5.html) — o formato da expressão de cronograma.
- [crontab.guru](https://crontab.guru/) — referência interativa de expressões cron.
- [systemd timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) — uma alternativa moderna com logging embutido.
- [Google SRE Book: Distributed Periodic Scheduling with Cron](https://sre.google/sre-book/distributed-periodic-scheduling/) — preocupações de confiabilidade de jobs agendados.
