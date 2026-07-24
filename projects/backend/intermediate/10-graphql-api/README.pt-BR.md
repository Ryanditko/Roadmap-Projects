# API GraphQL com Resolvers

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa uma API onde o cliente, e não o servidor, decide o formato da resposta. Em vez de um conjunto fixo de endpoints REST, você expõe um único endpoint e um schema tipado; os clientes pedem exatamente os campos de que precisam e recebem exatamente isso — nem mais, nem menos. A engenharia interessante está por baixo: resolvers que buscam cada campo, o notório problema N+1 que aparece no instante em que você resolve uma lista, e o batching (dataloaders) que o resolve. Você também enfrentará o outro lado da flexibilidade do GraphQL — uma única query maliciosa pode pedir o mundo, então limites de profundidade e complexidade se tornam uma preocupação real.

## Pré-requisitos

- Conforto para construir uma API HTTP e consultar um banco de dados
- Familiaridade com ao menos um conjunto de dados que tenha relacionamentos (ex.: autores → posts → comentários)
- Entendimento básico de tipos e schemas
- Uma biblioteca de servidor GraphQL para sua linguagem (Apollo, graphql-js, Strawberry, gqlgen, etc.)
- Consciência de como o REST lida com os mesmos problemas, para contraste

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um schema GraphQL com tipos, queries, mutations e modificadores non-null/lista
- Escrever resolvers que buscam dados para cada campo, incluindo relacionamentos aninhados
- Reconhecer o problema de query N+1 e eliminá-lo com um dataloader de batching
- Implementar paginação baseada em cursor sobre um campo de lista
- Aplicar autorização dentro dos resolvers com base no usuário autenticado
- Proteger o servidor com limites de profundidade e complexidade de query

## Requisitos Funcionais

1. A API deve expor um único endpoint que aceita queries e mutations GraphQL.
2. O schema deve definir ao menos dois tipos relacionados e resolver campos aninhados sob demanda.
3. Um campo que o cliente não pediu nunca deve aparecer na resposta.
4. Resolver uma lista de pais seguida de um campo filho não deve disparar uma query por pai (sem N+1).
5. Ao menos um campo de lista deve suportar paginação baseada em cursor retornando informações de página.
6. As mutations devem validar a entrada e retornar erros tipados para dados inválidos.
7. Os resolvers devem aplicar autorização, escondendo ou rejeitando campos que o usuário atual não pode acessar.
8. O servidor deve rejeitar queries que excedam um orçamento configurado de profundidade ou complexidade.

## Marcos Sugeridos

1. **Marco 1 — Schema e resolvers:** Defina tipos e escreva resolvers que respondem queries aninhadas contra sua fonte de dados.
2. **Marco 2 — Batching e paginação:** Adicione um dataloader para matar o N+1, depois adicione paginação por cursor a uma lista.
3. **Marco 3 — Mutations e proteções:** Adicione mutations validadas, autorização no nível do resolver e limites de profundidade/complexidade.

## Esboço de Dados e Interface

```text
type Author {
  id: ID!
  name: String!
  posts(first: Int, after: String): PostConnection!
}
type Post { id: ID!  title: String!  author: Author! }

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}
type PageInfo { hasNextPage: Boolean!  endCursor: String }

type Query    { author(id: ID!): Author  posts: [Post!]! }
type Mutation { createPost(input: CreatePostInput!): CreatePostPayload! }

Armadilha N+1: posts -> para cada post, resolver .author => N queries de autor
Correção:      agrupar todos os ids de autor em uma chamada via dataloader
Proteções:     rejeitar se profundidade > 6 ou custo > orçamento
```

## Desafios Extras

- Adicione uma subscription (ex.: `postAdded`) via WebSockets para atualizações em tempo real.
- Implemente cache no nível de campo ou persisted queries para reduzir trabalho repetido.
- Adicione uma diretiva customizada (ex.: `@auth(role: ADMIN)`) aplicada durante a execução.
- Exponha o custo da query em uma extensão da resposta para que os clientes se auto-ajustem.

## Definição de Pronto

- [ ] Um cliente pode pedir um subconjunto de campos e recebe precisamente esses campos, profundamente aninhados.
- [ ] Resolver uma lista mais um campo relacionado dispara um número limitado de queries, verificado por log.
- [ ] Uma lista paginada retorna cursores estáveis e `hasNextPage` correto entre páginas.
- [ ] Entrada inválida de mutation retorna um erro tipado e acionável em vez de um 500.
- [ ] Uma query profunda ou complexa demais é rejeitada antes da execução, não depois.

## Armadilhas Comuns

- Ignorar o problema N+1 até que uma query de lista se ramifique em centenas de chamadas ao banco.
- Colocar lógica de negócio/autorização na camada de transporte em vez dos resolvers, de modo que ela seja contornada.
- Retornar `null` para um campo non-null (`!`), o que anula todo o objeto pai inesperadamente.
- Paginação por offset em dados mutáveis, causando itens pulados ou duplicados entre páginas.
- Nenhum limite de complexidade, deixando uma query profundamente aninhada virar uma negação de serviço acidental.

## Recursos

- [GraphQL: Learn oficial](https://graphql.org/learn/) — schema, queries, resolvers e execução na fonte.
- [Apollo: Entendendo o problema N+1](https://www.apollographql.com/docs/technotes/TN0019-avoiding-the-n-plus-1/) — por que acontece e como dataloaders resolvem.
- [Spec de Cursor Connections do GraphQL](https://relay.dev/graphql/connections.htm) — o formato padrão para paginação por cursor.
- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html) — limites de profundidade/complexidade e outras proteções.
