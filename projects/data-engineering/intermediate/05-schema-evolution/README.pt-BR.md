# Sistema de Evolução de Schema

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um sistema que permite a um schema de dados mudar ao longo do tempo sem quebrar os produtores e consumidores que dependem dele. Pipelines reais nunca congelam seu schema: campos são adicionados, renomeados, depreciados e ocasionalmente removidos — e ainda assim os dados do ano passado precisam continuar legíveis e o consumidor da semana passada não pode falhar com os registros de hoje. Você implementará um schema registry que versiona schemas, classifica cada mudança proposta como retrocompatível, compatível-adiante ou totalmente compatível, e rejeita mudanças quebradoras antes que entrem em produção. Depois você escreverá migrações que atualizam registros antigos para o schema atual na leitura. A lição é que schema é um contrato, e evoluir um contrato com segurança é uma disciplina, não um detalhe posterior.

## Pré-requisitos

- Conforto para modelar dados com um formato de serialização (JSON Schema, Avro ou Protobuf)
- Entendimento de produtores e consumidores compartilhando um contrato de dados
- Familiaridade com conceitos de versionamento semântico
- O [projeto de streaming Kafka](../03-streaming-kafka/) é um contexto útil de por que a compatibilidade importa

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Versionar schemas e armazená-los em um registry chaveado por assunto (subject) e versão
- Distinguir compatibilidade retro, adiante e total e raciocinar sobre quem quebra quando
- Detectar uma mudança quebradora (ex.: remover um campo obrigatório) antes de ela ser registrada
- Migrar registros históricos para o schema atual na leitura
- Gerenciar a depreciação de campos com defaults em vez de remoção abrupta

## Requisitos Funcionais

1. O registry deve armazenar múltiplas versões de um schema sob um nome de assunto estável.
2. Registrar uma nova versão deve rodar uma checagem de compatibilidade contra a(s) versão(ões) anterior(es).
3. Uma mudança retro-incompatível (ex.: remover/renomear um campo obrigatório) deve ser rejeitada com um motivo claro.
4. Adicionar um campo opcional com default deve ser aceito como uma mudança retrocompatível.
5. Um leitor deve conseguir desserializar um registro escrito com qualquer versão registrada no schema mais recente.
6. Campos depreciados devem ser suportados via defaults para que registros antigos e novos leiam corretamente.
7. Toda versão registrada deve ser recuperável por número para que dados históricos permaneçam decodificáveis.

## Marcos Sugeridos

1. **Marco 1 — Registry versionado:** Armazene e recupere schemas por assunto e número de versão.
2. **Marco 2 — Checagens de compatibilidade:** Classifique uma mudança proposta e rejeite as quebradoras automaticamente.
3. **Marco 3 — Migração na leitura:** Atualize registros de qualquer versão antiga para a mais recente na leitura.

## Esboço de Dados e Interface

```text
registry:
  subject "user"
    v1: { id: int, name: string }
    v2: { id: int, name: string, email: string = "" }   # +opcional, default -> retrocompatível
    v3: { id: int, full_name: string }                    # renomear obrigatório -> REJEITADO

compatibility(nova, antiga) -> BACKWARD | FORWARD | FULL | NONE
  adicionar opcional c/ default    -> BACKWARD ok
  remover/renomear campo obrigat.  -> NONE     -> rejeita
  adicionar obrigatório sem default -> NONE    -> rejeita

read(record):
  detecta writer_version do record
  aplica migrações v_writer -> v_latest (preenche defaults, mapeia renomeações)
  retorna record no formato v_latest
```

## Desafios Extras

- Suporte um *modo* de compatibilidade por assunto (backward / forward / full) e aplique-o de acordo.
- Adicione um relatório dry-run "o que quebraria" para um schema proposto antes de registrá-lo.
- Trate uma renomeação controlada via um alias que lê tanto o nome antigo quanto o novo do campo.
- Integre com Avro ou Protobuf e reutilize suas regras nativas de resolução.

## Definição de Pronto

- [ ] Schemas são armazenados e recuperáveis por assunto e versão, com um histórico imutável.
- [ ] Adicionar um campo opcional com default é aceito; remover um campo obrigatório é rejeitado com motivo.
- [ ] Um registro escrito sob a v1 desserializa corretamente contra o schema mais recente.
- [ ] Campos depreciados resolvem via defaults sem erros de leitura.
- [ ] O classificador de compatibilidade é coberto por testes para cada tipo de mudança.

## Armadilhas Comuns

- Tratar "adicionar um campo" como sempre seguro — um novo campo *obrigatório* sem default quebra produtores antigos.
- Mutar uma versão registrada no lugar em vez de criar uma nova, corrompendo o histórico.
- Confundir compatibilidade retro e adiante e aplicar a direção errada para sua ordem de rollout.
- Renomear um campo e chamar isso de compatível; para os leitores é uma remoção mais uma adição.
- Confiar na ordem ou posição dos campos em vez de nomes, fazendo qualquer reordenação ler dados errados silenciosamente.

## Recursos

- [Confluent: Evolução e compatibilidade de schema](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html) — a taxonomia definitiva de compatibilidade.
- [Apache Avro: Resolução de Schema](https://avro.apache.org/docs/current/specification/#schema-resolution) — como leitores e escritores reconciliam schemas.
- [Documentação do JSON Schema](https://json-schema.org/understanding-json-schema/) — modelando e validando schemas em evolução.
- [Protocol Buffers: Atualizando um tipo de mensagem](https://protobuf.dev/programming-guides/proto3/#updating) — regras de número de campo para evolução segura.
