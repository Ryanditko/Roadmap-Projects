# Construtor Visual de Páginas (arrastar/soltar)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um editor de arrastar e soltar onde não desenvolvedores compõem páginas soltando componentes em um canvas, aninhando-os, editando suas propriedades e visualizando o resultado ao vivo — o padrão por trás do Webflow, do Framer e do editor de blocos do Notion. A interação de arrastar enganosamente simples esconde o desafio real: um modelo de documento. Cada página é uma árvore serializável de nós, e cada ação (soltar, mover, editar, excluir) é uma transformação bem definida dessa árvore que deve percorrer perfeitamente até o armazenamento, suportar desfazer/refazer e renderizar tanto um canvas editável quanto uma saída limpa. Este projeto é fundamentalmente sobre modelar uma árvore e construir operações previsíveis sobre ela, com o arrastar e soltar como a superfície visível.

## Pré-requisitos

- Domínio forte de composição de componentes e gerenciamento de estado
- Conforto para modelar estruturas de dados recursivas/de árvore
- Entendimento da API de Drag and Drop do HTML ou de uma abordagem de arraste baseada em ponteiro
- Familiaridade com padrões de atualização imutável para estado previsível

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar uma página como uma árvore de componentes serializável com um schema de nó estável
- Implementar arrastar e soltar que insere e reordena nós, incluindo aninhamento
- Construir um editor de propriedades que edita o nó selecionado e reflete mudanças ao vivo
- Implementar desfazer/refazer como operações inversas sobre a árvore do documento
- Serializar para e desserializar do armazenamento para que uma página salva recarregue identicamente

## Requisitos Funcionais

1. Usuários devem arrastar componentes de uma paleta para um canvas e vê-los renderizar imediatamente.
2. Componentes devem ser aninháveis (um container pode conter filhos) com alvos de soltar válidos/inválidos claros.
3. Selecionar um componente deve abrir um editor de propriedades cujas mudanças atualizam o canvas ao vivo.
4. Toda ação que muta deve ser desfazível e refazível na ordem correta.
5. A página deve serializar para um formato portável e desserializar de volta para uma árvore idêntica.
6. O construtor deve renderizar uma prévia/saída limpa que corresponde ao canvas sem o chrome do editor.
7. Reordenar e mover nós deve manter a árvore válida (sem ciclos, sem nós órfãos).

## Marcos Sugeridos

1. **Marco 1 — Árvore e renderização:** Defina o schema de nó e renderize uma árvore estática em um canvas.
2. **Marco 2 — Arrastar, soltar, aninhar:** Implemente o arraste da paleta e a reordenação/aninhamento no canvas.
3. **Marco 3 — Propriedades e prévia:** Adicione o editor de propriedades e um modo de prévia sem chrome.
4. **Marco 4 — Histórico e persistência:** Adicione desfazer/refazer e serialização/desserialização para o armazenamento.

## Esboço de Dados e Interface

```text
   Árvore do documento (serializável):
   {
     id, type: "Page",
     children: [
       { id, type: "Container", props: {...}, children: [
           { id, type: "Text",  props: { content } },
           { id, type: "Image", props: { src, alt } }
       ]}
     ]
   }

   Paleta ──arrasta──▶ Canvas (renderiza árvore) ──seleciona──▶ Editor de props
                         │                                        │
                         └──── mutações (inserir/mover/editar/excluir) ──┐
                                                                         ▼
                                          Histórico: [op, op-inversa] p/ desfazer/refazer

Operações (puras, árvore -> árvore):
  insert(parentId, index, node)   move(nodeId, newParentId, index)
  updateProps(nodeId, patch)      remove(nodeId)
Invariantes:  sem ciclos · todo nó alcançável · ids únicos
```

## Desafios Extras

- Adicione prévia responsiva em múltiplos breakpoints com overrides de props por breakpoint.
- Adicione snap-to-grid e guias de alinhamento para posicionamento preciso.
- Suporte templates/símbolos de componentes reutilizáveis que atualizam todas as instâncias.
- Adicione edição multiusuário opcional sobre o modelo de árvore.

## Definição de Pronto

- [ ] Um componente arrastado da paleta renderiza no canvas e pode ser aninhado em um container.
- [ ] Editar uma propriedade no painel atualiza o canvas imediatamente.
- [ ] Desfazer e refazer revertem e reproduzem corretamente todo tipo de mutação.
- [ ] Salvar e depois recarregar reproduz exatamente a mesma árvore de página.
- [ ] A saída de prévia corresponde ao canvas sem elementos exclusivos do editor vazando.

## Armadilhas Comuns

- Armazenar a página como HTML renderizado em vez de uma árvore estruturada, tornando edição e desfazer intratáveis.
- Mutar a árvore no lugar, tornando desfazer/refazer e a detecção de mudança não confiáveis.
- Permitir soltar inválidos (um container dentro de seu próprio descendente) que criam ciclos e travam a renderização.
- Acoplar o editor de propriedades ao código concreto do componente, de modo que adicionar um componente signifique editar o painel.
- Vazar wrappers exclusivos do editor (contornos de seleção, alças de arraste) para a saída serializada.

## Recursos

- [MDN: HTML Drag and Drop API](https://developer.mozilla.org/pt-BR/docs/Web/API/HTML_Drag_and_Drop_API) — o modelo nativo de interação de arraste.
- [Documentação do dnd kit](https://docs.dndkit.com/) — um toolkit de arraste moderno e acessível para construir UIs ordenáveis/aninháveis.
- [MDN: JSON](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/JSON) — serializando e restaurando a árvore do documento.
- [Redux: Implementing Undo History](https://redux.js.org/usage/implementing-undo-history) — um modelo claro para desfazer/refazer sobre estado.
