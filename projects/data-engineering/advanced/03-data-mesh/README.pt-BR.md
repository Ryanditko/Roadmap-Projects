# Simulação de Data Mesh

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Simule um data mesh: em vez de um time central dono de um data warehouse monolítico, vários domínios independentes publicam cada um seus dados como um **data product** com dono, contrato versionado, SLOs de qualidade e uma entrada descobrível em um catálogo. Você modelará dois ou três domínios (digamos, Pedidos, Pagamentos e Clientes), dará a cada um uma porta de saída publicada e imporá um **contrato de dados** para que uma mudança de schema quebradora seja capturada antes de chegar aos consumidores. A parte difícil é o organizacional transformado em técnico: como manter domínios autônomos e ainda garantir que um consumidor consiga fazer join entre eles? Você projetará governança federada — regras globais que todos seguem, liberdade local em tudo o mais — e a provará com uma consulta entre domínios que só funciona porque os contratos se mantiveram.

## Pré-requisitos

- Experiência construindo pipelines de dados e pensando em produtores vs consumidores
- Familiaridade com formatos de schema e evolução (Avro, Protobuf ou JSON Schema)
- Entendimento do conceito de catálogo de dados / store de metadados
- Conforto com os tradeoffs da posse descentralizada (este é tanto um exercício de arquitetura quanto de código)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar um domínio como um data product autocontido com uma porta de saída clara
- Definir e versionar um contrato de dados e detectar mudanças quebradoras vs compatíveis
- Projetar governança federada: quais regras são globais, quais são locais
- Tornar data products descobríveis por um catálogo com posse e SLOs
- Raciocinar sobre consistência entre domínios sem um único time dono

## Requisitos Funcionais

1. Cada domínio deve expor ao menos um data product com schema documentado, dono e SLO de atualidade/qualidade.
2. Todo data product deve ser registrado em um catálogo compartilhado que um consumidor possa buscar por domínio, dono ou campo.
3. Um contrato publicado deve ser validado a cada nova versão do dataset; uma mudança incompatível para trás deve ser rejeitada ou sinalizada antes da publicação.
4. Um consumidor deve conseguir rodar uma consulta que faz join dos produtos de dois domínios usando apenas seus contratos públicos.
5. Regras globais de governança (ex.: todo produto carrega um `data_owner` e uma classificação de PII) devem ser impostas uniformemente.
6. Violações de contrato e quebras de SLO devem aparecer para o domínio dono, não para um time central.

## Marcos Sugeridos

1. **Marco 1 — Domínios e produtos:** Modele 2–3 domínios, cada um publicando um data product com schema, dono e metadados de SLO.
2. **Marco 2 — Contratos e catálogo:** Adicione validação de contrato na publicação e registre tudo em um catálogo pesquisável.
3. **Marco 3 — Governança federada:** Imponha regras globais, conecte alertas de SLO/contrato aos donos de domínio e demonstre um join entre domínios.

## Esboço de Dados e Interface

```text
Domínio "orders"                   Domínio "payments"
  data product: orders_daily         data product: settlements_daily
  dono: orders-team@example.com      dono: payments-team@example.com
  contrato v2 { orderId, amount,     contrato v1 { orderId, settledAt,
    currency, dt }  slo: freshness<2h  status }  slo: completeness>99.9%
        │  publicar (validar contrato)        │
        └──────────────┬───────────────────────┘
                       ▼
             [catálogo / metadados]
               registry[product] = { schema, dono, slo, piiClass, versão }
               search(field="orderId") -> [orders_daily, settlements_daily]
                       │
                       ▼
   consulta do consumidor: JOIN orders_daily ON settlements_daily USING(orderId)
   (funciona só porque ambos os contratos expõem orderId de forma compatível)

Regras globais (federadas): todo produto DEVE ter data_owner + piiClass.
Liberdade local: storage, ferramentas de transformação, modelo interno por domínio.
```

## Desafios Extras

- Adicione diffing automático de contratos no CI que bloqueia um merge introduzindo uma mudança quebradora.
- Implemente um "registro de consumidor" para que produtores saibam quem depende deles antes de mudar um schema.
- Adicione testes de qualidade de dados por produto (taxas de nulos, integridade referencial) que alimentam o status do SLO.

## Definição de Pronto

- [ ] Cada domínio publica um data product de posse independente com schema, dono e SLO.
- [ ] O catálogo é pesquisável e retorna posse e SLO para qualquer produto.
- [ ] Uma mudança de schema incompatível para trás é capturada no momento da publicação, não por um consumidor quebrado.
- [ ] Um join entre domínios roda usando apenas contratos públicos.
- [ ] Campos globais de governança são impostos em todo produto; a ausência de um bloqueia a publicação.

## Armadilhas Comuns

- Reconstruir um warehouse central com passos extras — domínios que não conseguem publicar sem um time central não são autônomos.
- Tratar "contrato" como documentação em vez de uma verificação imposta e versionada.
- Nenhuma política de mudança quebradora, então toda edição de schema arrisca silenciosamente os jobs a jusante.
- Um catálogo que ninguém atualiza — a descoberta decai no momento em que o registro é manual e opcional.

## Recursos

- [Princípios do Data Mesh (Zhamak Dehghani)](https://martinfowler.com/articles/data-mesh-principles.html) — os quatro princípios fundadores.
- [Data Mesh: Arquitetura Lógica](https://martinfowler.com/articles/data-monolith-to-mesh.html) — domínios e produtos explicados.
- [Confluent Schema Registry: Compatibilidade](https://docs.confluent.io/platform/current/schema-registry/fundamentals/avro.html) — como os modos de compatibilidade formalizam contratos.
- [OpenLineage](https://openlineage.io/docs/) — um padrão para descrever datasets e seus produtores/consumidores.
