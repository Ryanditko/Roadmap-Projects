# Interface de Notas (Local Storage)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um app de anotações: uma lista de notas de um lado, um editor do outro, e tudo salvo no `localStorage` para sobreviver a um recarregamento. É a lista de tarefas amadurecida — em vez de itens de uma linha, você tem documentos titulados e editáveis, e em vez de uma única ação, você tem criar-ler-atualizar-excluir completo. O desafio central é o padrão mestre-detalhe: uma lista e um editor que ficam sincronizados, onde selecionar uma nota a carrega, editar a atualiza ao vivo e excluir limpa sem deixar uma seleção órfã. Adicione busca e salvamento automático e começa a parecer uma ferramenta de verdade.

## Pré-requisitos

- Fundamentos de HTML, CSS e JavaScript
- A API `localStorage` e serialização `JSON`
- Manipulação de arrays/objetos para operações CRUD
- Básico de debouncing para salvamento automático (ou um temporizador)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar CRUD completo sobre uma coleção persistida no `localStorage`
- Construir um layout mestre-detalhe onde uma lista e um editor ficam sincronizados
- Salvar edições automaticamente com debouncing em vez de um botão de salvar manual
- Buscar/filtrar notas por título ou conteúdo em tempo real
- Tratar casos limítrofes: nenhuma nota, excluir a nota selecionada, títulos vazios

## Requisitos Funcionais

1. O usuário pode criar uma nova nota, que aparece na lista e abre no editor.
2. Editar o título ou o corpo de uma nota atualiza a lista e persiste automaticamente.
3. O usuário pode excluir uma nota, com uma confirmação, e a seleção é resetada de forma sensata.
4. Uma caixa de busca filtra a lista de notas por título ou conteúdo.
5. As notas persistem entre recarregamentos via `localStorage`.
6. Cada nota mostra um carimbo de data/hora de última atualização que reflete edições reais.
7. Um estado vazio é mostrado quando não há notas ou nenhuma correspondência de busca.

## Marcos Sugeridos

1. **Marco 1 — Lista e criação:** Renderize a lista de notas e suporte criar e selecionar notas.
2. **Marco 2 — Editar e persistir:** Edite título/corpo, salve automaticamente no `localStorage`, atualize os carimbos de data/hora.
3. **Marco 3 — Excluir e buscar:** Adicione exclusão com confirmação e busca ao vivo sobre as notas.

## Esboço de Dados e Interface

```text
Note
  id:        string   (crypto.randomUUID())
  title:     string
  body:      string
  updatedAt: number   (Date.now())

Estado do app
  notes:      Note[]
  selectedId: string | null
  query:      string

Layout (mestre-detalhe)
+------------------+-----------------------------+
| [ busca........ ]| Título [ ................ ] |
| + Nova nota      |                             |
|------------------|  textarea do corpo          |
| > Compras     ·  |  ...                        |
|   Reunião     ·  |                             |
|   Ideias      ·  |  atualizado há 2m  [Excluir]|
+------------------+-----------------------------+
```

## Desafios Extras

- Adicione fixação para que notas importantes fiquem no topo da lista.
- Adicione tags ou pastas e filtre por elas.
- Suporte renderização básica de Markdown em um painel de pré-visualização.
- Adicione exportar/importar todas as notas como um único arquivo JSON.

## Definição de Pronto

- [ ] Criar, editar e excluir atualizam a lista, o editor e os dados armazenados.
- [ ] As edições são salvas automaticamente sem uma ação manual de salvar.
- [ ] Excluir a nota selecionada deixa uma seleção sensata ou um estado vazio.
- [ ] A busca filtra a lista em tempo real e mostra um estado vazio quando nada corresponde.
- [ ] Todas as notas sobrevivem intactas a um recarregamento da página.

## Armadilhas Comuns

- Escrever no `localStorage` a cada tecla sem debouncing, causando travamentos.
- Perder a seleção atual ou travar quando a nota selecionada é excluída.
- Manter o valor do editor fora de sincronia com a nota armazenada após uma troca.
- Exceder a cota do `localStorage` com notas grandes e não tratar o erro.
- Esquecer o estado vazio, fazendo um app recém-aberto parecer quebrado.

## Recursos

- [MDN: Web Storage API](https://developer.mozilla.org/pt-BR/docs/Web/API/Web_Storage_API) — persistir e ler dados estruturados.
- [MDN: JSON.stringify()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) — serializar sua coleção de notas.
- [CSS-Tricks: Debouncing and Throttling](https://css-tricks.com/debouncing-throttling-explained-examples/) — limitar a taxa do salvamento automático.
- [web.dev: Learn Forms — textarea](https://web.dev/learn/forms/textareas) — entrada de múltiplas linhas acessível.
