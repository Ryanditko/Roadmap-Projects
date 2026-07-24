# Pipeline de Dados Otimizado para Custo

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Pegue um pipeline de dados funcional mas perdulário e corte sua conta de nuvem sem quebrar seus SLAs. Este é um projeto de otimização em que a função objetivo é dólares, e a disciplina é medir o custo por execução *antes* de tocar em qualquer coisa. Você atribuirá o gasto entre compute, armazenamento e transferência de dados, e então atacará o maior item: instâncias spot/preemptíveis para trabalho batch tolerante a falhas, tiering de armazenamento e compressão, evitar egress entre regiões, right-sizing de clusters superdimensionados, e agendar jobs não urgentes em janelas mais baratas. A tensão constante é custo vs performance e custo vs confiabilidade — uma instância spot é barata até ser recuperada no meio do job. A entrega é uma análise de custo antes/depois com a economia de cada otimização e seu trade-off de risco documentados.

## Pré-requisitos

- Um stack de dados em nuvem que você possa medir (Spark gerenciado, um warehouse, ou object storage + motor de consulta)
- Entendimento das dimensões de preço de nuvem: horas de compute, classe de armazenamento, egress, requisições
- Familiaridade com instâncias spot/preemptíveis e seu comportamento de recuperação
- Conforto para ler um detalhamento de custo/faturamento e atribuí-lo a cargas de trabalho

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Instrumentar e atribuir o custo do pipeline entre compute, armazenamento e transferência
- Usar capacidade spot/preemptível para trabalho tolerante a falhas sem arriscar perda de dados
- Aplicar tiering de armazenamento, compressão e layout de arquivos para cortar custos de armazenamento e varredura
- Eliminar cobranças evitáveis de transferência/egress de dados
- Pesar a economia de cada otimização contra seu risco de performance e confiabilidade

## Requisitos Funcionais

1. O custo por execução do pipeline deve ser medido e atribuído a compute, armazenamento e transferência antes da otimização.
2. Ao menos um estágio tolerante a falhas deve rodar em capacidade spot/preemptível com tratamento seguro da recuperação (checkpoint/retry).
3. O custo de armazenamento deve ser reduzido via tiering e/ou compressão, com os dados ainda consultáveis dentro do SLA.
4. Uma mudança documentada deve eliminar ou reduzir o custo de transferência entre regiões/egress.
5. Toda otimização deve preservar a correção do pipeline e seu SLA de latência/atualidade.
6. Um alerta de orçamento/custo deve disparar quando o gasto exceder um limite definido.

## Marcos Sugeridos

1. **Marco 1 — Medir e atribuir:** Instrumente o custo por execução e decomponha-o por recurso; identifique os 2–3 maiores itens.
2. **Marco 2 — Otimizar compute e armazenamento:** Mova um estágio para spot com checkpointing; aplique tiering/compressão; meça de novo.
3. **Marco 3 — Transferência e guardrails:** Corte egress, adicione agendamento em janelas baratas e conecte um alerta de orçamento.

## Esboço de Dados e Interface

```text
atribuição de custo (por execução, antes):
  compute  $X  (horas de cluster x preço da instância)   <- geralmente o maior
  storage  $Y  (GB-mês x classe de armazenamento)
  transfer $Z  (egress entre regiões GB x taxa)
  total    $T  --> alvo: reduzir para $T' no mesmo SLA

otimizações e risco:
  on-demand -> spot (batch)   economiza ~60-90%   risco: recuperação -> precisa checkpoint+retry
  standard  -> storage tiered economiza no frio    risco: latência de recuperação no archive
  entre regiões -> mesma região economiza egress   risco: menos geo-redundância
  right-size do cluster        economiza ocioso     risco: menos folga para burst
  agendar fora de pico         economiza (mkt spot) risco: conclusão mais tarde

guardrail: se gasto_no_mês > ORÇAMENTO -> alerta (não estourar silenciosamente)
invariante: correção + SLA inalterados após cada mudança.
```

## Desafios Extras

- Adicione um gráfico de Pareto custo-vs-latência para que um stakeholder possa escolher um ponto, não só "o mais barato".
- Implemente right-sizing automático que escala o cluster à carga observada por execução.
- Modele descontos de uso comprometido / reservado e calcule a utilização de break-even.

## Definição de Pronto

- [ ] O custo por execução é medido e atribuído antes e depois, com a economia total quantificada.
- [ ] Um estágio apoiado em spot sobrevive à recuperação via checkpoint/retry sem perda de dados.
- [ ] O custo de armazenamento e/ou varredura cai mensuravelmente enquanto os dados seguem consultáveis dentro do SLA.
- [ ] Uma otimização de egress/transferência é documentada com sua economia.
- [ ] Um alerta de orçamento dispara na quebra do limite; o risco de cada otimização é anotado.

## Armadilhas Comuns

- Otimizar um custo que você não consegue ver — sem atribuição, você corta a coisa errada.
- Colocar um estágio com estado e não recuperável em spot e perder trabalho quando ele é recuperado.
- Comprimir/tierar tão agressivamente que uma consulta estoura seu SLA de latência buscando dados frios.
- Ignorar o egress até que a linha de transferência domine a conta — leituras entre regiões são dinheiro silencioso.

## Recursos

- [AWS Well-Architected: Pilar de Otimização de Custo](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) — um framework estruturado para decisões de custo.
- [AWS: Boas práticas de Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) — usar capacidade interruptível com segurança.
- [Google Cloud: Classes de armazenamento](https://cloud.google.com/storage/docs/storage-classes) — economia de tiering e tradeoffs de recuperação.
- [Spark: Tuning](https://spark.apache.org/docs/latest/tuning.html) — right-sizing de recursos para não pagar por ociosidade.
