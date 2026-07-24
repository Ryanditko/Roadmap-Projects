# Sistema de Logging Centralizado

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um pipeline que coleta logs de vários serviços em execução, os envia para um armazenamento central e os torna pesquisáveis em um só lugar. Em vez de acessar três máquinas via SSH e dar grep em arquivos, você deve conseguir responder "mostre todo erro do serviço de pagamentos na última hora" a partir de uma única interface de consulta. Escolha uma stack — ELK/EFK (Elasticsearch + Kibana), ou Grafana Loki, ou equivalente — e monte coleta, parsing, armazenamento com política de retenção e dashboards. O tema é transformar texto disperso e não estruturado em dados consultáveis, correlacionados e limitados.

## Pré-requisitos

- Dois ou mais serviços (ou contêineres) que emitam logs, idealmente em formatos diferentes
- Docker / Docker Compose ou um pequeno cluster para hospedar a stack de logging
- Entendimento de stdout/stderr, níveis de log e logs estruturados vs texto puro
- Familiaridade básica com consultas e JSON

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Enviar logs de múltiplas fontes com um coletor/agente (Fluent Bit, Fluentd, Filebeat, Promtail)
- Fazer parsing e enriquecer logs em campos estruturados (nível, serviço, timestamp, trace id)
- Guardar logs em um backend pesquisável e consultar todas as fontes de uma vez
- Aplicar uma política de retenção para que o armazenamento não cresça sem limite
- Construir um dashboard e ao menos um alerta sobre um sinal derivado de logs

## Requisitos Funcionais

1. Logs de ao menos dois serviços distintos devem ser coletados sem alterar o código da app além da saída estruturada.
2. Um coletor deve encaminhar logs ao armazenamento central, com buffer para evitar perda durante backpressure.
3. Logs devem ser transformados em campos estruturados, incluindo nível, nome do serviço e timestamp.
4. Uma única consulta deve conseguir filtrar entre todos os serviços por campo (ex.: `level=error AND service=payments`).
5. Uma política de retenção deve automaticamente descartar ou arquivar logs mais antigos que uma janela definida.
6. Um dashboard deve visualizar o volume de logs e a taxa de erros ao longo do tempo.
7. Um alerta deve disparar quando a taxa de erros cruzar um limite.

## Marcos Sugeridos

1. **Marco 1 — Coletar e guardar:** Leve os logs dos serviços ao backend central, visíveis em bruto.
2. **Marco 2 — Parsear e consultar:** Estruture os logs em campos e consulte entre fontes.
3. **Marco 3 — Reter e observar:** Adicione retenção, um dashboard e um alerta por limite.

## Esboço de Dados e Interface

```text
Fluxo:
  serviço A ─┐
  serviço B ─┼─> coletor/agente ─(buffer)─> store/índice ─> UI de consulta + dashboards
  serviço C ─┘                                   |
                                            job de retenção (expiração)

Registro de log estruturado (formato alvo):
  timestamp:  ISO-8601
  level:      DEBUG|INFO|WARN|ERROR
  service:    string
  message:    string
  trace_id:   string (opcional, para correlação)

Exemplo de consulta (conceitual):
  service = "payments" AND level = "ERROR" AND time > now-1h
```

## Desafios Extras

- Correlacione logs entre serviços usando um trace/request id compartilhado.
- Adicione multitenância ou acesso por papel para que times vejam apenas seus próprios logs.
- Envie métricas derivadas de logs (ex.: contagem de erros) para um sistema de métricas.
- Adicione camadas de ciclo de vida de índice/stream (hot/warm/cold) para controlar o custo de armazenamento.

## Definição de Pronto

- [ ] Logs de todas as fontes aparecem no armazenamento central segundos após serem emitidos.
- [ ] Uma única consulta filtra por nível e serviço em todas as fontes.
- [ ] Logs são estruturados, não guardados como blobs opacos de texto.
- [ ] Logs antigos são removidos ou arquivados automaticamente conforme a política de retenção.
- [ ] Um dashboard mostra a taxa de erros e um alerta dispara quando ela sobe.

## Armadilhas Comuns

- Coletar texto bruto sem parsing, permitindo grep mas nunca agregação ou alerta.
- Sem buffer no coletor, uma indisponibilidade do armazenamento descarta logs silenciosamente.
- Esquecer a retenção, deixando o índice crescer até esgotar o disco e derrubar o armazenamento.
- Timestamps/fusos inconsistentes entre serviços, tornando a correlação enganosa.
- Registrar segredos ou dados pessoais em um armazenamento pesquisável sem mascaramento.

## Recursos

- [Documentação do Grafana Loki](https://grafana.com/docs/loki/latest/) — agregação de logs projetada para ser econômica.
- [Documentação do Elastic Stack (ELK)](https://www.elastic.co/guide/index.html) — Elasticsearch, Logstash, Kibana.
- [Documentação do Fluent Bit](https://docs.fluentbit.io/manual) — coleta e parsing leves de logs.
- [App de Doze Fatores: Logs](https://12factor.net/pt_br/logs) — por que apps devem tratar logs como fluxos de eventos.
</content>
