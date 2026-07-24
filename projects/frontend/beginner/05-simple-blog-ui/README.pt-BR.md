# Interface de Blog Simples

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa o lado de leitura de um blog: uma lista de posts que você pode navegar, buscar e filtrar por categoria, mais uma visão de detalhe para ler um único post por completo. Não há escrita nem backend — os posts vêm de um arquivo JSON local ou de um pequeno array em memória — então o foco fica em apresentar o conteúdo com clareza e transitar entre uma lista e um detalhe sem recarregar a página inteira. É aqui que iniciantes encontram pela primeira vez a ideia de uma "view": os mesmos dados renderizados de duas formas, e uma navegação simples no cliente que mantém a URL e a tela em acordo.

## Pré-requisitos

- Fundamentos de HTML, CSS e JavaScript
- Métodos de array (`filter`, `map`, `find`) para transformações de listas
- Entendimento básico de roteamento no cliente ou alternância mostrar/ocultar de views
- Um framework de componentes à sua escolha é opcional, mas bem-vindo

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Renderizar uma lista de itens de uma fonte de dados e uma visão de detalhe para um item
- Implementar busca no cliente e filtragem por categoria sobre o mesmo conjunto de dados
- Navegar entre as views de lista e detalhe mantendo o estado consistente
- Paginar ou revelar preguiçosamente uma lista longa sem sobrecarregar o DOM
- Apresentar tipografia legível e uma ordem de leitura acessível

## Requisitos Funcionais

1. A view inicial lista posts com título, resumo, categoria e tempo estimado de leitura.
2. Selecionar um post abre uma view de detalhe mostrando seu conteúdo completo.
3. Uma caixa de busca filtra a lista comparando o texto do título ou do resumo.
4. Filtros de categoria restringem a lista; limpá-los restaura o conjunto completo.
5. Uma lista longa é paginada ou usa "carregar mais" em vez de renderizar tudo de uma vez.
6. O usuário pode retornar de uma view de detalhe para a lista sem perder seu filtro.
7. Cada detalhe de post tem um único `h1` e uma estrutura lógica de títulos.

## Marcos Sugeridos

1. **Marco 1 — Lista e detalhe:** Carregue os posts e renderize a lista, depois uma view de detalhe completa na seleção.
2. **Marco 2 — Busca e filtro:** Adicione busca por texto e filtragem por categoria sobre o conjunto de dados.
3. **Marco 3 — Navegação e paginação:** Preserve filtros entre views e pagine a lista.

## Esboço de Dados e Interface

```text
Post
  id:          string
  title:       string
  excerpt:     string
  body:        string
  category:    string
  publishedAt: string   (ISO-8601)
  readMinutes: number

Views
  list   -> array filtrado/paginado de resumos de Post
  detail -> um Post por id

Layout (lista)               Layout (detalhe)
+-------------------------+   +----------------------+
| [ busca ] [Categoria v] |   | < Voltar             |
+-------------------------+   | Título (h1)          |
| Cartão de post          |   | meta: cat · 5 min    |
| Cartão de post          |   |                      |
| Cartão de post          |   | parágrafos...        |
+-------------------------+   +----------------------+
| < 1 2 3 >               |
+-------------------------+
```

## Desafios Extras

- Sincronize a view atual e os filtros com a URL (query params ou hash) para que os links sejam compartilháveis.
- Adicione uma nuvem de tags e cruze links de posts relacionados por tags compartilhadas.
- Adicione um sumário gerado a partir dos títulos do post.
- Mostre um estado vazio de "nenhum resultado" com uma forma de resetar os filtros.

## Definição de Pronto

- [ ] As views de lista e detalhe renderizam os mesmos dados corretamente de uma fonte.
- [ ] Busca e filtros de categoria se combinam e podem ser limpos.
- [ ] Retornar à lista preserva a busca e o filtro ativos.
- [ ] Uma lista longa não renderiza todos os posts no DOM de uma vez.
- [ ] Cada página de detalhe tem exatamente um `h1` e títulos ordenados.

## Armadilhas Comuns

- Duplicar os dados do post para a lista e o detalhe em vez de derivar ambos de uma fonte.
- Perder o filtro ativo ao voltar de uma view de detalhe.
- Busca sensível a maiúsculas que perde correspondências óbvias.
- Renderizar centenas de cartões de imediato, deixando a página travada.
- Esquecer um estado vazio, fazendo uma lista totalmente filtrada parecer um bug.

## Recursos

- [MDN: Array.prototype.filter()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) — construir busca e filtragem por categoria.
- [MDN: History API](https://developer.mozilla.org/pt-BR/docs/Web/API/History_API) — refletir views na URL.
- [web.dev: Learn Accessibility — Content structure](https://web.dev/learn/accessibility/structure) — títulos e ordem de leitura.
- [MDN: Trabalhando com JSON](https://developer.mozilla.org/pt-BR/docs/Learn/JavaScript/Objects/JSON) — carregar e analisar dados locais.
