# Plataforma de Blog com CRUD e Comentários

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa a API por trás de uma plataforma de blog: autores escrevem posts, leitores deixam comentários e tudo é organizado por tags e um fluxo de rascunho/publicação. O que eleva isto acima de um CRUD comum são os relacionamentos — comentários que se aninham em respostas, posts pertencentes a seus autores e moderação que oculta conteúdo sem destruí-lo. Você vai lidar com recursos aninhados, paginação sobre listas potencialmente grandes e verificações de permissão que respondem de forma limpa "este usuário pode editar aquele post?" em toda requisição.

## Pré-requisitos

- Conforto para construir recursos REST ([API REST Simples de Tarefas](../../beginner/01-simple-rest-api-task-management/) é um bom aquecimento)
- Autenticação básica e sessões ou tokens ([API de Autenticação Básica](../../beginner/03-basic-authentication-api/))
- Um banco com suporte a relações ou referências (PostgreSQL, MySQL ou MongoDB)
- Entendimento de relacionamentos um-para-muitos e chaves estrangeiras

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar recursos aninhados (posts → comentários → respostas) em uma API REST
- Implementar paginação e raciocinar sobre os trade-offs entre offset e cursor
- Aplicar permissões de propriedade e de papel em operações de escrita
- Implementar soft delete para que o conteúdo possa ser ocultado e recuperado depois
- Filtrar e buscar posts por tag de forma eficiente

## Requisitos Funcionais

1. Autores devem poder criar, atualizar e excluir seus próprios posts; outros devem ser proibidos com 403.
2. Um post deve ter status `draft` ou `published`; rascunhos não devem aparecer em listagens públicas.
3. Qualquer usuário autenticado deve poder comentar em um post publicado e responder a comentários existentes.
4. Listagens de comentários devem retornar respostas aninhadas (ou resolvíveis) sob seu pai.
5. Listagens de posts devem ser paginadas e suportar filtro por uma ou mais tags.
6. Excluir um post ou comentário deve ser um soft delete — o registro é ocultado, não removido fisicamente.
7. Um moderador/admin deve poder ocultar (rejeitar) qualquer comentário independentemente da autoria.

## Marcos Sugeridos

1. **Marco 1 — CRUD de posts:** Criar, ler, atualizar e excluir posts com verificação de propriedade e o status rascunho/publicado.
2. **Marco 2 — Comentários e aninhamento:** Adicione comentários e respostas aninhadas, retornando-os em uma árvore legível.
3. **Marco 3 — Descoberta:** Filtro por tag, busca e listagens paginadas de posts e comentários.
4. **Marco 4 — Moderação e soft delete:** Oculte/recupere posts e comentários e exponha um endpoint de moderação.

## Esboço de Dados e Interface

```text
Post     id, authorId, title, body, tags[], status, createdAt, deletedAt?
Comment  id, postId, authorId, parentId?, body, hidden, createdAt, deletedAt?

POST /posts                 (autor)   -> 201 post
GET  /posts?tag=go&page=2             -> 200 { items, page, total }
PATCH /posts/{id}           (dono)    -> 200 post | 403
POST /posts/{id}/comments             -> 201 comentário
GET  /posts/{id}/comments             -> 200 [ { ...comentário, replies: [...] } ]
DELETE /comments/{id}       (soft)    -> 204
POST /comments/{id}/hide    (mod)     -> 200

Paginação: ?page=&limit=  (ou cursor: ?after=<id>)
```

## Desafios Extras

- Adicione um recurso de seguir para que um usuário veja um feed de posts dos autores que segue.
- Rastreie contagens de visualização e exponha uma listagem "mais lidos da semana".
- Suporte Markdown no corpo dos posts, renderizando para HTML sanitizado.
- Adicione sugestões de posts relacionados com base em tags compartilhadas.

## Definição de Pronto

- [ ] Um usuário não consegue editar ou excluir um post ou comentário que não é seu (a menos que moderador).
- [ ] Posts em rascunho nunca aparecem em nenhuma listagem pública/anônima.
- [ ] Respostas são corretamente associadas ao comentário pai e retornadas aninhadas.
- [ ] Conteúdo soft-deleted desaparece das leituras mas continua recuperável no armazenamento.
- [ ] Listagens são paginadas e o filtro por tag retorna apenas posts correspondentes.

## Armadilhas Comuns

- Carregar todos os comentários e montar a árvore de respostas em memória a cada requisição — pagine e limite a profundidade.
- Esquecer de excluir linhas soft-deleted de toda query, fazendo o conteúdo "excluído" reaparecer.
- Verificar a propriedade depois de executar a escrita em vez de antes dela.
- Vazar posts em rascunho por um endpoint de busca ou tag que pula o filtro de status.
- Paginação por offset que desloca resultados quando novos posts são inseridos durante a rolagem.

## Recursos

- [MDN: Códigos de status de resposta HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status) — escolha o código certo para proibido, não-encontrado e sem-conteúdo.
- [PostgreSQL: Documentação](https://www.postgresql.org/docs/current/) — referência para modelar e filtrar linhas.
- [Use The Index, Luke: Paginação](https://use-the-index-luke.com/no-offset) — por que a paginação por cursor supera a por offset em escala.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — onde modelagem de dados e APIs se encaixam no geral.
