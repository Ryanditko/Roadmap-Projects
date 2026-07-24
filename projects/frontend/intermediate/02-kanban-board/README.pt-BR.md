# Quadro Kanban (Arrastar e Soltar)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um quadro Kanban — colunas como *A Fazer*, *Em Progresso* e *Concluído* contendo cartões arrastáveis — que uma pessoa possa reorganizar pegando um cartão e soltando em outra coluna. A parte satisfatória é o arrastar em si, mas o aprendizado real está por baixo: modelar o quadro como estado normalizado, mover um cartão entre duas listas sem corromper a ordem, persistir o layout para que sobreviva a um reload e tornar tudo isso operável sem mouse. Arrastar e soltar é notoriamente inacessível quando feito de forma ingênua, então um caminho por teclado e leitor de tela é um requisito de primeira classe aqui, não algo secundário.

## Pré-requisitos

- Conforto para construir componentes e gerenciar estado em um framework (React, Vue, Svelte ou Angular)
- Entendimento de arrays e atualizações imutáveis (mover um item entre duas listas)
- Familiaridade com o modelo de eventos do navegador (eventos de ponteiro, drag ou teclado)
- Uso básico de `localStorage` para persistência
- Opcional: conhecer uma biblioteca de arrastar e soltar (dnd kit, SortableJS) versus a API nativa HTML Drag and Drop

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar um quadro como estado normalizado (colunas, cartões e uma ordem explícita) em vez de arrays aninhados
- Implementar o movimento de um cartão dentro e entre colunas sem perdê-lo ou duplicá-lo
- Escolher entre a API nativa Drag and Drop e uma biblioteca, justificando o trade-off
- Fornecer uma alternativa por teclado ao arrastar (pegar, mover, soltar)
- Anunciar ações de reordenação e movimento à tecnologia assistiva
- Persistir e reidratar o estado do quadro entre reloads da página

## Requisitos Funcionais

1. O quadro deve renderizar múltiplas colunas, cada uma com uma lista ordenada de cartões.
2. Um cartão deve ser arrastável de uma coluna e solto em qualquer coluna numa posição escolhida.
3. A reordenação deve atualizar a ordem subjacente de forma determinística — sem cartões duplicados ou perdidos.
4. O usuário deve poder criar, editar e excluir cartões, e a edição deve abrir um formulário de detalhe do cartão.
5. O estado completo do quadro deve persistir em `localStorage` e restaurar no reload.
6. Toda interação de arrastar deve ter um equivalente por teclado (pegar, mover entre colunas, soltar).
7. Os alvos de soltura devem mostrar um indicador de drop claro, e os movimentos devem ser anunciados via uma região ARIA live.

## Marcos Sugeridos

1. **Marco 1 — Quadro estático:** Renderize colunas e cartões a partir de estado normalizado; adicione criar/editar/excluir cartão.
2. **Marco 2 — Arrastar para mover:** Implemente arrastar e soltar por ponteiro dentro e entre colunas com um indicador de drop.
3. **Marco 3 — Persistir:** Salve e reidrate o estado do quadro a partir do `localStorage`.
4. **Marco 4 — Teclado e a11y:** Adicione um caminho de movimento por teclado e anúncios em região live.

## Esboço de Dados e Interface

```text
Layout
  ┌──────────┬──────────────┬──────────┐
  │ A Fazer  │ Em Progresso │Concluído │
  ├──────────┼──────────────┼──────────┤
  │ [card 1] │ [card 4]     │ [card 6] │
  │ [card 2] │ [card 5]     │          │
  │ [card 3] │  ▁▁drop▁▁    │          │  ← indicador de drop
  │ [+ add]  │ [+ add]      │ [+ add]  │
  └──────────┴──────────────┴──────────┘

Estado (normalizado)
  columns:     { id, title, cardIds: string[] }[]   // a ordem vive aqui
  cards:       Record<id, { id, title, description, priority }>
  dragging:    { cardId, fromColumn } | null

Card  { id, title, description, priority: 'low'|'med'|'high' }

Operação de mover (pura)
  move(state, cardId, toColumn, toIndex) -> novo estado
  // remove o id dos cardIds de origem, insere no destino em toIndex
```

## Desafios Extras

- Adicione filtragem de cartões ou uma caixa de busca que esmaece cartões que não correspondem.
- Suporte múltiplos quadros, alternáveis por uma barra lateral.
- Adicione desfazer/refazer para ações de mover e excluir.
- Adicione uma visão de raia por "prioridade" que agrupa cartões independentemente da coluna.
- Sincronize o estado do quadro com uma API de backend em vez de `localStorage`.

## Definição de Pronto

- [ ] Cartões movem dentro e entre colunas com a ordem preservada e sem duplicatas.
- [ ] O quadro sobrevive a um reload completo com colunas e ordem dos cartões intactas.
- [ ] Todo movimento possível por arrastar também é possível apenas por teclado.
- [ ] Um indicador de drop mostra onde o cartão vai cair antes da soltura.
- [ ] Ações de mover e reordenar são anunciadas a leitores de tela.

## Armadilhas Comuns

- Armazenar cartões aninhados dentro das colunas, tornando movimentos entre colunas uma cópia profunda propensa a erros — normalize.
- Depender só de eventos nativos de drag, que são inutilizáveis por teclado e instáveis em telas de toque.
- Mutar o array de ordem no lugar, fazendo o React/Vue perder a mudança ou renderizar ordem obsoleta.
- Esquecer chaves estáveis, fazendo cartões pularem visualmente ou perderem o foco durante uma reordenação.
- Tratar acessibilidade como desafio extra — adaptar um caminho de teclado a um drag só de ponteiro é doloroso.

## Recursos

- [MDN: API HTML Drag and Drop](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) — as primitivas nativas e seus limites.
- [Documentação do dnd kit](https://docs.dndkit.com/) — um toolkit de arrastar e soltar acessível e amigável ao teclado.
- [W3C ARIA APG: Regiões live](https://www.w3.org/WAI/ARIA/apg/practices/live-regions/) — anunciando movimentos dinâmicos.
- [web.dev: Acesso por teclado](https://web.dev/articles/keyboard-access) — tornando interações operáveis sem mouse.
- [MDN: Window.localStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/localStorage) — persistindo o quadro.
