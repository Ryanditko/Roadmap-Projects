# Projete um Sistema de Colaboração em Tempo Real

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete um editor colaborativo em tempo real no estilo do Google Docs, onde dezenas de pessoas editam o mesmo documento simultaneamente e cada tecla converge para um único estado consistente em todas as telas — com cursores, presença e edições offline que se reconciliam na reconexão. O coração do problema é o controle de concorrência sobre dados mutáveis compartilhados: duas pessoas digitando na mesma posição precisam mesclar deterministicamente sem um lock e sem perder nenhuma edição. Isso força uma escolha entre duas famílias de algoritmos, Operational Transformation e CRDTs. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Entendimento de consistência eventual e o conceito de convergência
- Familiaridade com transporte em tempo real (WebSockets) e pub/sub
- Domínio conceitual de Operational Transformation (OT) e CRDTs
- Consciência dos desafios da sincronização offline-first

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Comparar as abordagens OT e CRDT e justificar uma escolha para um editor de texto
- Projetar uma garantia de convergência para que todas as réplicas cheguem ao mesmo estado independentemente da ordem
- Modelar presença, cursores e seleções como estado efêmero (não persistido)
- Tratar edição offline e reconciliação sem perder ou duplicar edições
- Raciocinar sobre o custo de armazenar histórico de edições e como compactá-lo

## Requisitos e Restrições

1. Todos os clientes editando um documento devem convergir para conteúdo idêntico.
2. Edições concorrentes na mesma posição devem mesclar deterministicamente, sem lock, sem perda de dados.
3. Refletir edições remotas com latência percebida sub-200 ms para usuários co-localizados.
4. Suportar edição offline que se reconcilia corretamente na reconexão.
5. Manter presença por usuário e posições de cursor como estado transitório.
6. Preservar um histórico de versões e permitir restaurar versões anteriores.
7. Escalar para documentos com muitos editores concorrentes e grandes históricos de edição.

## Abordagem Sugerida

Escolha primeiro o modelo de concorrência, porque tudo o mais decorre disso. **OT** transforma cada operação contra as concorrentes e tradicionalmente depende de um servidor central para ordenar operações — armazenamento mais simples, mas as funções de transformação são notoriamente delicadas. **CRDTs** anexam identidade a cada caractere/elemento para que as mesclagens sejam comutativas e não precisem de autoridade central — metadados mais ricos por caractere, mais armazenamento, mas robustos offline. Roteie edições por um servidor por documento (ou shard) que faz broadcast para os assinantes via pub/sub. Mantenha presença/cursores em um canal efêmero separado que nunca é persistido. Para offline, acumule operações locais e reproduza/mescle na reconexão. Gerencie o histórico com snapshots periódicos mais um log de operações, compactando operações antigas atrás de um snapshot.

## Esboço de Arquitetura

```text
Cliente A ─op─┐
Cliente B ─op─┼─> Servidor de sessão do doc (por doc / sharded) ─> Ordenação + merge OT/CRDT
Cliente C ─op─┘        │                                            │
                       ├─broadcast(op)─> assinantes                 └─> Log de ops + snapshots periódicos (store)
                       └─canal de presença (efêmero: cursores, seleções) NÃO persistido

Offline: cliente acumula ops -> reconecta -> envia acumuladas -> servidor mescla -> converge

APIs principais (sobre WebSocket):
JOIN   { docId, sinceVersion }     -> { snapshot, version }
OP     { docId, op, baseVersion }  -> ACK { version }  (+ broadcast para outros)
PRESENCE { docId, cursorPos, selection }   # efêmero, best-effort

Modelo de dados (esboço):
Doc{ id, snapshot, version, opLog[] }       # opLog compactado atrás de snapshots
Op{ type: INSERT|DELETE, pos/id, char, siteId, seq }  # CRDT: id estável por caractere
```

## Tópicos de Aprofundamento

- **OT vs CRDT:** garantias de convergência, custo de metadados, robustez offline, complexidade.
- **Prova de convergência:** por que ops concorrentes mesclam para o mesmo estado independentemente da ordem de chegada.
- **Presença como estado efêmero:** canal separado, nunca persistido, entrega best-effort.
- **Reconciliação offline:** buffering de ops, ordenação causal, tombstones para deleções.
- **Compactação de histórico:** snapshots mais log de ops; garbage collection de operações antigas.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com o modelo de merge e arquitetura de sessão, refinado.
- [ ] Uma escolha fundamentada OT-vs-CRDT com a garantia de convergência declarada.
- [ ] O fluxo de reconciliação de edição offline especificado de ponta a ponta.
- [ ] Uma análise de falha/DR: crash do servidor de sessão, edições em split-brain, corrupção do log de ops.
- [ ] Um plano de histórico/armazenamento mostrando cadência de snapshots e compactação do log de ops.

## Armadilhas Comuns

- Usar last-write-wins no documento inteiro, descartando silenciosamente edições concorrentes.
- Persistir dados de presença/cursor, inchando o armazenamento com estado transitório.
- Assumir que operações chegam em ordem; o merge deve ser correto para qualquer ordem de chegada.
- Esquecer tombstones para deleções em um CRDT, fazendo uma deleção e um insert concorrente divergirem.
- Manter um log de ops ilimitado sem snapshots, tornando o carregamento do documento mais lento com o tempo.

## Recursos

- [Conflict-free Replicated Data Types (artigo CRDT)](https://inria.hal.science/inria-00609399/document) — a fundação formal dos CRDTs.
- [Operational Transformation (Wikipedia)](https://en.wikipedia.org/wiki/Operational_transformation) — o modelo OT e sua história na edição colaborativa.
- [Yjs](https://docs.yjs.dev/) — um framework CRDT amplamente usado que vale estudar para design do mundo real.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de tempo real e pub/sub.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — convergência e resolução de conflitos.
