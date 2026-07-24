# Pipeline de Ingestão de Múltiplas Fontes

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um pipeline que puxa o mesmo tipo de entidade — digamos, clientes ou produtos — de várias fontes muito diferentes e as mescla em um único dataset limpo e consistente. Uma fonte é uma API REST, outra é um envio de CSV, outra uma exportação de banco de dados; cada uma nomeia seus campos de forma diferente, discorda nos formatos e, às vezes, descreve o mesmo registro do mundo real. Seu trabalho é esconder essa bagunça atrás de uma interface de conector comum, mapear cada fonte para um schema compartilhado e resolver conflitos quando duas fontes divergem. Essa é a realidade de integração por trás de todo projeto de "visão única do cliente", e a lição é que a parte difícil nunca é a busca — é a reconciliação.

## Pré-requisitos

- Conforto para chamar uma API HTTP e ler arquivos (CSV/JSON) programaticamente
- Entendimento de mapeamento de schema e normalização
- Familiaridade com deduplicação e a ideia de uma chave natural/de negócio
- Qualquer linguagem, mais duas ou três fontes (reais ou simuladas) descrevendo registros sobrepostos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma interface de conector que abstrai fontes heterogêneas
- Mapear os campos e formatos de cada fonte para um único schema alvo compartilhado
- Deduplicar registros que representam a mesma entidade entre fontes
- Resolver conflitos com uma estratégia explícita de precedência ou recência
- Isolar falhas para que uma fonte quebrada não afunde a execução inteira

## Requisitos Funcionais

1. Cada fonte deve ser acessada por meio de uma interface de conector comum (fetch → registros brutos).
2. Toda fonte deve mapear para um único schema compartilhado com tipos e formatos normalizados.
3. O pipeline deve deduplicar registros entre fontes usando uma chave natural definida.
4. Quando fontes divergem sobre um campo, uma regra documentada de resolução de conflito deve decidir o vencedor.
5. Uma falha em uma fonte não deve abortar a ingestão das outras; ela deve ser registrada e reportada.
6. As credenciais/config de cada fonte devem ser externalizadas, não hardcoded.
7. O pipeline deve emitir estatísticas por fonte: buscados, mapeados, rejeitados, deduplicados.

## Marcos Sugeridos

1. **Marco 1 — Conectores:** Defina a interface de conector e implemente duas fontes por trás dela.
2. **Marco 2 — Normalizar e mesclar:** Mapeie cada fonte para o schema compartilhado e deduplique pela chave natural.
3. **Marco 3 — Conflitos e resiliência:** Adicione resolução de conflitos, isolamento de erro por fonte e estatísticas.

## Esboço de Dados e Interface

```text
Connector (interface)
  name() -> string
  fetch() -> iterable<raw_record>
  map(raw_record) -> canonical_record | reject(motivo)

canonical_record (schema compartilhado)
  entity_id     string   (chave natural, ex.: email normalizado name@example.com)
  full_name     string
  email         string
  updated_at    timestamp
  _source       string
  _fetched_at   timestamp

merge:
  agrupar por entity_id
  em conflito -> escolher por precedência [db > api > csv]  OU  updated_at mais recente

source_stats: source | fetched | mapped | rejected | merged
```

## Desafios Extras

- Adicione uma etapa de correspondência fuzzy para que nomes/emails quase duplicados colapsem em uma entidade.
- Torne cada conector reexecutável de forma independente, com backoff em erros transitórios de API.
- Rastreie a proveniência em nível de campo para responder qual fonte forneceu cada valor.
- Suporte busca incremental por fonte (apenas registros alterados desde a última execução).

## Definição de Pronto

- [ ] Adicionar uma nova fonte exige apenas uma nova implementação de conector, sem mudanças no pipeline.
- [ ] Duas fontes descrevendo a mesma entidade produzem um registro mesclado, não dois.
- [ ] Um conflito de campo resolve de forma determinística conforme a regra documentada.
- [ ] Uma fonte retornando erro ainda deixa as outras concluírem, com a falha reportada.
- [ ] As estatísticas por fonte fecham: buscados = mapeados + rejeitados.

## Armadilhas Comuns

- Embutir lógica específica de fonte no núcleo do pipeline em vez de atrás da interface de conector.
- Deduplicar em um campo bruto (email sem normalização/caixa) de modo que duplicatas reais escapem.
- Resolver conflitos implicitamente pela ordem de iteração do último a escrever — torne a regra explícita.
- Deixar o timeout ou a falha de autenticação de uma fonte derrubar a execução inteira.
- Perder de qual fonte veio um valor, tornando a depuração posterior impossível.

## Recursos

- [Airbyte: Desenvolvimento de conectores](https://docs.airbyte.com/connector-development/) — como um sistema em produção modela fontes plugáveis.
- [Especificação Singer](https://github.com/singer-io/getting-started/blob/master/docs/SPEC.md) — um padrão aberto para conectores de fonte/destino.
- [Wikipedia: Record linkage](https://en.wikipedia.org/wiki/Record_linkage) — a teoria por trás de casar registros entre fontes.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — capítulos sobre integração de dados e evolução de schema.
