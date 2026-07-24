# Front-end de E-commerce

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa o front-end voltado ao comprador de uma loja online: navegar por um catálogo, refiná-lo com filtros e busca, inspecionar um produto, adicionar itens ao carrinho e percorrer um checkout que valida antes de enviar. Não há processador de pagamento real aqui — o objetivo é a maquinaria do lado do cliente da qual uma loja depende. Você vai equilibrar estado de servidor (produtos buscados de uma API) contra estado de cliente (o carrinho), manter o carrinho vivo entre reloads da página e rotear entre as visões de catálogo, detalhe e checkout. É o projeto onde "só um pouquinho de estado" cresce em algo que precisa de uma estratégia de verdade.

## Pré-requisitos

- Conforto para construir componentes e elevar estado em um framework moderno (React, Vue, Svelte ou Angular)
- Buscar dados de uma API REST e renderizar resultados assíncronos
- Fundamentos de roteamento no cliente (como uma URL mapeia para uma visão)
- Familiaridade com formulários e inputs controlados
- Entendimento de `localStorage` para persistência

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Separar estado de servidor (catálogo) de estado de cliente (carrinho) e gerenciar cada um adequadamente
- Implementar filtragem, ordenação e busca que se compõem sem brigar entre si
- Persistir o carrinho entre reloads e reconciliá-lo com dados de produto atualizados
- Construir um formulário de checkout com múltiplos campos, validação por campo e mensagens de erro claras
- Tratar estados de carregamento, vazio e erro para cada visão orientada a dados
- Tornar as interações de catálogo e carrinho acessíveis por teclado e leitor de tela

## Requisitos Funcionais

1. O catálogo deve buscar produtos de uma API e renderizá-los em uma grade responsiva.
2. O usuário deve poder filtrar por categoria e preço, buscar por palavra-chave e ordenar (preço, avaliação, mais recentes) — e esses controles devem se combinar.
3. Cada produto deve ter uma visão de detalhe alcançável por sua própria URL, para poder ser compartilhada e favoritada.
4. Adicionar um produto ao carrinho deve atualizar um selo de carrinho visível; as quantidades devem ser ajustáveis e os itens removíveis.
5. O carrinho deve sobreviver a um reload completo da página e recalcular totais a partir dos preços atuais dos produtos.
6. O checkout deve validar todo campo obrigatório antes de permitir o envio e exibir erros inline.
7. Toda visão de dados deve mostrar um estado de carregamento, um estado vazio e um estado de erro recuperável.

## Marcos Sugeridos

1. **Marco 1 — Catálogo:** Busque e renderize a grade de produtos com estados de carregamento e erro.
2. **Marco 2 — Filtrar, buscar, ordenar:** Sobreponha controles componíveis ao catálogo e reflita-os na URL.
3. **Marco 3 — Detalhe e carrinho:** Adicione páginas de produto roteadas e um carrinho persistente com controles de quantidade.
4. **Marco 4 — Checkout:** Construa o formulário de checkout validado e uma tela de confirmação de pedido.

## Esboço de Dados e Interface

```text
Layout
  [ Cabeçalho: logo | busca | selo do carrinho(3) ]
  [ Filtros laterais ][ Grade de produtos          ]
  Rotas: /  /product/:id  /cart  /checkout  /confirmation

Estado
  products:  Product[]        (estado de servidor, buscado)
  cart:      CartItem[]       (estado de cliente, persistido)
  filters:   { q, category, minPrice, maxPrice, sort }  (espelhado na URL)

Product   { id, title, price, category, rating, image, stock }
CartItem  { productId, qty }   // guarde só a qty; preço lido do produto

API consumida
  GET /api/products?category=&q=&sort=  -> 200 Product[]
  GET /api/products/:id                 -> 200 Product | 404
```

## Desafios Extras

- Adicione uma lista de desejos que persiste separadamente do carrinho.
- Mostre "produtos relacionados" na página de detalhe, da mesma categoria.
- Faça debounce no input de busca e cancele requisições obsoletas em andamento.
- Adicione atualizações otimistas de quantidade que revertem em caso de falha.
- Suporte um código de cupom que ajusta o total do carrinho.

## Definição de Pronto

- [ ] Filtros, busca e ordenação se combinam corretamente e são refletidos na URL.
- [ ] O carrinho persiste entre reloads e os totais recalculam a partir dos preços vigentes.
- [ ] O checkout bloqueia o envio até que todos os campos obrigatórios sejam válidos, com erros inline.
- [ ] Estados de carregamento, vazio e erro existem para as visões de catálogo e detalhe.
- [ ] Todos os controles interativos são alcançáveis e operáveis por teclado.

## Armadilhas Comuns

- Armazenar objetos de produto completos (incluindo preço) no carrinho, deixando os preços obsoletos após uma atualização do catálogo.
- Tratar filtros como flags independentes que se sobrescrevem em vez de compor uma única query.
- Esquecer o estado vazio — uma grade em branco após um filtro parece um bug.
- Validar todo o formulário de checkout só no envio, deixando o usuário caçar o único campo errado.
- Perder o foco quando uma gaveta modal de carrinho abre ou fecha.

## Recursos

- [MDN: Usando a Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch) — fundamentos de busca de dados.
- [web.dev: Patterns](https://web.dev/patterns/) — soluções reutilizáveis para problemas comuns de UI, incluindo formulários e carregamento.
- [MDN: Validação de formulário no cliente](https://developer.mozilla.org/pt-BR/docs/Learn/Forms/Form_validation) — construindo uma validação de checkout confiável.
- [MDN: Window.localStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/localStorage) — persistindo o carrinho.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — onde gerência de estado e roteamento se encaixam no quadro geral.
