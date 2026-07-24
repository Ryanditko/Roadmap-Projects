# API de E-commerce com Autenticação JWT

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa o backend de uma loja online: um catálogo de produtos, um carrinho de compras por usuário e um fluxo de pedidos que transforma um carrinho em um pedido pago e com estoque ajustado. A autenticação usa JWTs, e o acesso é controlado por papel — um cliente pode navegar e comprar, um admin pode gerenciar o catálogo. A dificuldade interessante aqui não é um endpoint isolado, mas como eles se compõem: reservar estoque sem vender além do disponível, manter o checkout atômico e emitir tokens que expiram corretamente. É aqui que o CRUD deixa de ser suficiente e o pensamento transacional real começa.

## Pré-requisitos

- Uma base de API REST ([API REST Simples de Tarefas](../../beginner/01-simple-rest-api-task-management/) é uma boa base)
- Noções de autenticação por token ([API de Autenticação Básica](../../beginner/03-basic-authentication-api/) cobre o fundamento)
- Um banco relacional ou de documentos e seu modelo de transações (PostgreSQL, MySQL ou MongoDB)
- Entender hashing de senhas e assinatura/verificação de JWTs

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Emitir, verificar e expirar tokens de acesso JWT e desenhar uma estratégia de refresh
- Aplicar controle de acesso por papel (RBAC) no nível da rota e do recurso
- Modelar produtos, carrinhos, pedidos e usuários com relacionamentos corretos
- Envolver o checkout em uma transação de banco para manter estoque e pedidos consistentes
- Evitar venda além do estoque sob compras concorrentes usando locking ou atualizações atômicas

## Requisitos Funcionais

1. Usuários devem poder se registrar e fazer login, recebendo um JWT assinado em caso de sucesso.
2. Requisições a endpoints protegidos devem ser rejeitadas com 401 quando o token estiver ausente, expirado ou inválido.
3. Endpoints exclusivos de admin (criar/editar/excluir produto) devem retornar 403 para tokens não-admin.
4. Um usuário deve poder adicionar produtos ao carrinho, atualizar quantidades e remover itens.
5. O checkout deve criar um pedido, decrementar o estoque e esvaziar o carrinho como uma única operação atômica.
6. Um pedido nunca deve ser criado para uma quantidade maior que o estoque disponível.
7. Usuários devem poder listar seus próprios pedidos anteriores e ver o detalhe de um pedido.
8. A listagem de produtos deve suportar filtro por categoria e paginação.

## Marcos Sugeridos

1. **Marco 1 — Auth e papéis:** Registro, login, emissão de JWT e middleware que verifica tokens e aplica papéis.
2. **Marco 2 — Catálogo e carrinho:** CRUD de produtos (admin) e operações de carrinho escopadas ao usuário autenticado.
3. **Marco 3 — Checkout e pedidos:** Criação transacional de pedido com decremento de estoque e histórico de pedidos.
4. **Marco 4 — Robustez sob concorrência:** Reproduza uma venda acima do estoque com requisições paralelas e corrija com locking ou decrementos atômicos.

## Esboço de Dados e Interface

```text
User    id, email, passwordHash, role ("customer" | "admin")
Product id, name, priceCents, stock, categoryId
Cart    userId, items: [{ productId, qty }]
Order   id, userId, status, totalCents, lines: [{ productId, qty, unitPriceCents }]

POST /auth/register  -> 201 { id, email }
POST /auth/login     -> 200 { accessToken, refreshToken }
GET  /products?category=&page=  -> 200 { items, page, total }
POST /products       (admin)    -> 201 { id, ... }   | 403
POST /cart/items     body: { productId, qty }        -> 200 cart
POST /orders/checkout            -> 201 pedido | 409 (estoque insuficiente)
GET  /orders         -> 200 [ ...pedidos próprios ]

Authorization: Bearer <jwt>   (claims: sub, role, exp)
```

## Desafios Extras

- Adicione rotação de refresh-token com revogação no servidor ao fazer logout.
- Suporte códigos promocionais/descontos aplicados no checkout com validação.
- Adicione avaliações de produto e uma nota agregada por produto.
- Emita um evento de confirmação de pedido (email mock ou webhook) após o checkout.

## Definição de Pronto

- [ ] Tokens expiram e tokens expirados são rejeitados com 401, não aceitos silenciosamente.
- [ ] Chamadores não-admin não conseguem criar, editar ou excluir produtos (verificado com teste).
- [ ] O checkout é atômico: uma falha no meio deixa estoque e carrinho inalterados.
- [ ] Checkouts concorrentes pela última unidade nunca têm sucesso simultâneo.
- [ ] Um usuário só vê seus próprios pedidos, nunca os de outro usuário.

## Armadilhas Comuns

- Confiar no payload do JWT sem verificar a assinatura — do contrário, qualquer um forja claims.
- Checar o estoque e decrementá-lo em duas queries separadas, deixando uma janela de corrida que vende além do disponível.
- Armazenar preços como float; use inteiros em centavos para evitar erros de arredondamento nos totais.
- Colocar verificações de papel apenas no cliente ou em algumas rotas, mas não em todas.
- Fazer o checkout como uma série de escritas independentes sem transação, deixando meio-pedido após uma falha.

## Recursos

- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519) — a especificação do JWT.
- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) — como aplicar RBAC corretamente.
- [PostgreSQL: Isolamento de Transações](https://www.postgresql.org/docs/current/transaction-iso.html) — a semântica na qual o checkout atômico se apoia.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — onde auth e bancos se encaixam no quadro maior.
