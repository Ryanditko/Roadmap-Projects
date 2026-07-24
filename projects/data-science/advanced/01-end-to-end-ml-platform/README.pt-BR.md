# Plataforma de ML Ponta a Ponta

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Projete e construa uma pequena plataforma interna de ML que leva um modelo dos dados brutos a um endpoint de produção monitorado sem nenhuma cola manual. A plataforma une quatro capacidades que normalmente ficam separadas: dados versionados, experimentos rastreados, um registro de modelos com estágios de ciclo de vida e um caminho automatizado de um modelo registrado até uma implantação de serving. O foco não é nenhum modelo específico — você pode treinar algo trivial — mas o encanamento que torna todo o ciclo reproduzível, auditável e re-executável meses depois por alguém que não o construiu. Esta é a espinha dorsal de MLOps que toda equipe de dados em produção eventualmente precisa.

## Pré-requisitos

- Conforto para treinar e avaliar modelos com um framework popular (scikit-learn, PyTorch ou TensorFlow)
- Experiência com containers e um runner de CI (Docker mais GitHub Actions ou similar)
- Familiaridade com serviços REST e armazenamento de objetos (S3/GCS ou um MinIO substituto)
- Ter construído ao menos alguns pipelines ponta a ponta antes (ex.: um projeto intermediário de pipeline de dados) ajuda bastante

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Versionar datasets e vincular cada artefato de modelo aos dados e ao código exatos que o produziram
- Rastrear experimentos (parâmetros, métricas, artefatos) e comparar execuções objetivamente
- Operar um registro de modelos com promoção por estágios (Staging → Produção → Arquivado)
- Automatizar a implantação para que uma promoção dispare um rollout de serving
- Instrumentar o ciclo com monitoramento e um gatilho de retreino

## Requisitos Funcionais

1. A plataforma deve versionar datasets para que qualquer modelo seja rastreável ao snapshot exato em que treinou.
2. Toda execução de treino deve registrar parâmetros, métricas e artefatos em um rastreador de experimentos consultável.
3. O registro de modelos deve armazenar modelos com metadados e suportar transições de estágio com trilha de auditoria.
4. Promover um modelo para Produção deve disparar automaticamente uma implantação de serving sem cópia manual de arquivos.
5. O endpoint de serving deve expor a versão atual do modelo em produção e o health via API.
6. O sistema deve registrar entradas/saídas de predição para análise posterior de monitoramento e drift.
7. Um job de retreino deve ser disparável tanto por agendamento quanto por um sinal de monitoramento.

## Requisitos Não Funcionais

- **Reprodutibilidade:** reexecutar o treino de um modelo registrado deve reproduzir as métricas dentro de uma tolerância documentada.
- **Disponibilidade:** o serving deve tolerar a falha de um único nó; meta de 99,5% para o endpoint.
- **Latência/vazão:** p95 do serving abaixo de um orçamento declarado (ex.: 200 ms) a uma taxa de requisições definida.
- **Auditabilidade:** cada promoção e implantação é atribuível a um usuário, versão de modelo e timestamp.

## Marcos Sugeridos

1. **Marco 1 — Rastreamento e versionamento de dados:** Suba rastreamento de experimentos (MLflow) e versionamento de datasets (DVC ou lakeFS); registre uma execução de treino de ponta a ponta.
2. **Marco 2 — Registro e promoção:** Registre modelos, implemente transições por estágios e grave a trilha de auditoria.
3. **Marco 3 — Serving automatizado:** Conecte um evento de promoção a uma implantação containerizada atrás de uma API estável.
4. **Marco 4 — Monitoramento e retreino:** Registre predições, calcule drift básico e feche o ciclo com um gatilho de retreino.

## Esboço de Dados e Interface

```text
                +-------------+     +------------------+
  dados brutos->| Versão Dados | -> | Job de Treino     |
                | (DVC/lakeFS) |    | logs -> Tracker   |
                +-------------+     +---------+--------+
                                              |
                                       registra modelo
                                              v
                                     +------------------+
                                     | Registro Modelos  |
                                     | Staging|Prod|Arq  |
                                     +---------+--------+
                                 promover(Prod)| evento
                                              v
                                     +------------------+     +-----------+
   cliente-> POST /predict -------> | Serving (ver Prod)| --> | log pred  |
             GET  /model/info       +------------------+     +-----+-----+
                                                                   |
                                          sinal drift/perf --------+--> retreino

Modelo registrado
  name, version, stage, run_id, data_version, metrics{}, created_by, created_at
```

## Desafios Extras

- Adicione uma feature store para que treino e serving compartilhem as mesmas definições de features.
- Suporte implantação shadow/canary: roteie uma fatia do tráfego para uma versão candidata.
- Adicione grafos de linhagem ligando dados → execução → modelo → implantação visualmente.
- Imponha portões de aprovação (um segundo revisor) antes da promoção para Produção.

## Definição de Pronto

- [ ] Qualquer modelo em produção rastreia limpo até sua versão de dados, commit de código e execução.
- [ ] Um evento de promoção implanta o modelo automaticamente, sem manuseio manual de artefatos.
- [ ] O endpoint de serving informa a versão de modelo ativa e passa nos health checks.
- [ ] Predições são registradas e uma métrica de drift é calculada de forma agendada.
- [ ] Uma execução de retreino pode ser disparada tanto por agendamento quanto por sinal de monitoramento.

## Armadilhas Comuns

- Tratar o rastreamento de experimentos como opcional e perder a capacidade de reproduzir o "melhor" modelo.
- Armazenar modelos como arquivos soltos em vez de entradas de registro, deixando o estado de ciclo de vida na cabeça de alguém.
- Acoplar o código de treino e de serving tão firmemente que uma mudança no serving força um retreino.
- Pular o registro de predições e depois não ter nada com que calcular drift quando a qualidade cai.

## Recursos

- [Documentação do MLflow](https://mlflow.org/docs/latest/index.html) — tracking, registro e empacotamento de modelos.
- [Google Cloud: MLOps — entrega contínua e pipelines de automação em ML](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) — a referência canônica de maturidade em MLOps.
- [Documentação do DVC](https://dvc.org/doc) — versionamento de dados e pipelines.
- [Hidden Technical Debt in Machine Learning Systems (Sculley et al., 2015)](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html) — por que a cola importa mais que o modelo.
