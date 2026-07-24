# Aplicativo de Lista de Tarefas

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa a clássica lista de tarefas: uma única tela onde o usuário digita uma tarefa, pressiona Enter, a vê aparecer, marca como concluída e a remove quando não importa mais. Parece trivial, mas é o menor exemplo completo do laço central do frontend — a entrada do usuário muda o estado, e o estado re-renderiza a UI. Tudo é conduzido por um array de objetos de tarefa em memória que você persiste no `localStorage` para que a lista sobreviva a um recarregamento. Não há servidor nem exigência de framework; o trabalho interessante é manter seu modelo de dados e o que o usuário vê perfeitamente sincronizados.

## Pré-requisitos

- HTML, CSS e JavaScript básicos (variáveis, arrays, funções)
- Como ler e escrever no DOM, ou um framework de componentes à sua escolha
- Familiaridade com métodos de array (`map`, `filter`, `find`)
- Um editor de código e um navegador com ferramentas de desenvolvedor

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar a UI como uma função de um único estado como fonte da verdade
- Adicionar, alternar, editar e excluir itens de forma imutável em vez de mutar no lugar
- Persistir e reidratar o estado com a API `localStorage`
- Renderizar uma visão filtrada (todas / ativas / concluídas) sem perder os dados subjacentes
- Conectar interações de teclado e clique com controles acessíveis e rotulados

## Requisitos Funcionais

1. O usuário pode digitar uma tarefa e adicioná-la pressionando Enter ou clicando em um botão Adicionar.
2. Tarefas vazias ou só com espaços devem ser rejeitadas sem adicionar uma linha em branco.
3. Cada tarefa pode ser alternada entre ativa e concluída, com uma diferença visual clara.
4. O usuário pode excluir qualquer tarefa individual.
5. Um contador ao vivo mostra quantas tarefas permanecem ativas.
6. O usuário pode filtrar a lista por todas, ativas ou concluídas.
7. A lista completa deve sobreviver a um recarregamento da página via `localStorage`.

## Marcos Sugeridos

1. **Marco 1 — Adicionar e renderizar:** Capture a entrada, adicione um objeto de tarefa ao estado, renderize a lista.
2. **Marco 2 — Alternar e excluir:** Marque tarefas como feitas e remova-as, atualizando o contador.
3. **Marco 3 — Filtrar e persistir:** Adicione a visão de filtro e salve/carregue do `localStorage`.

## Esboço de Dados e Interface

```text
Tarefa
  id:        string   (crypto.randomUUID())
  title:     string
  completed: boolean
  createdAt: number   (Date.now())

Layout
+------------------------------------------+
| [ nova tarefa............... ] [Adicionar]|
+------------------------------------------+
| [x] Comprar leite                 (del)  |
| [ ] Ligar para o dentista         (del)  |
+------------------------------------------+
| 1 item restante  [Todas][Ativas][Feitas] |
+------------------------------------------+
```

## Desafios Extras

- Edição em linha: dê um duplo clique no título de uma tarefa para renomeá-la.
- Adicione um botão "Limpar concluídas" que remove todas as tarefas feitas de uma vez.
- Permita reordenar tarefas com arrastar e soltar.
- Adicione um alternador de tema claro/escuro armazenado junto com as tarefas.

## Definição de Pronto

- [ ] Adicionar, alternar e excluir atualizam tanto a UI quanto o estado armazenado.
- [ ] Tarefas em branco não podem ser adicionadas.
- [ ] O contador de itens restantes está sempre correto após qualquer ação.
- [ ] Os filtros mudam a lista visível sem descartar tarefas ocultas.
- [ ] Recarregar a página restaura exatamente a lista anterior.

## Armadilhas Comuns

- Mutar o array de estado diretamente em vez de produzir um novo, causando renderizações desatualizadas.
- Armazenar strings de HTML renderizado em vez de um modelo de dados limpo, tornando filtros e edições penosos.
- Esquecer o `JSON.parse` / `JSON.stringify` em torno do `localStorage`, que só armazena strings.
- Usar o índice do array como chave/id, o que quebra após exclusão e reordenação.
- Pular rótulos nas caixas de seleção e botões, deixando o app inutilizável com um leitor de tela.

## Recursos

- [MDN: Web Storage API](https://developer.mozilla.org/pt-BR/docs/Web/API/Web_Storage_API) — como o `localStorage` funciona e seus limites.
- [MDN: Introdução ao DOM](https://developer.mozilla.org/pt-BR/docs/Web/API/Document_Object_Model/Introduction) — ler e atualizar a página.
- [web.dev: Learn Accessibility — Forms](https://web.dev/learn/accessibility/forms) — rotular entradas e controles.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — onde esses fundamentos se encaixam no quadro maior.
