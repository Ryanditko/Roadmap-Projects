# Editor de Markdown (Preview ao Vivo)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um editor de markdown com painel dividido: markdown bruto à esquerda, um preview HTML renderizado à direita, atualizando conforme o usuário digita. Parece simples até você notar as armadilhas — renderizar markdown não confiável é um vetor de XSS, reparsear a cada tecla trava em documentos grandes, e uma escrita ingênua em `innerHTML` descarta o cursor e a posição de rolagem do usuário. A lição real aqui é transformar texto em HTML seguro e sanitizado de forma eficiente, conectar uma barra de formatação e atalhos de teclado que editam o textarea de forma inteligente, e manter o editor responsivo à medida que o documento cresce. Você vai se apoiar em um parser de markdown de verdade em vez de reinventá-lo.

## Pré-requisitos

- Conforto com estado de componente e inputs controlados em um framework (React, Vue, Svelte ou Angular)
- Entendimento do DOM e do porquê definir `innerHTML` a partir de input do usuário é perigoso
- Familiaridade com APIs de seleção de textarea (`selectionStart`/`selectionEnd`)
- Conhecimento básico de debounce e de por que ele ajuda a performance de digitação
- Um parser de markdown (marked, markdown-it) e um sanitizador (DOMPurify) à sua escolha

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Parsear markdown para HTML com uma biblioteca mantida em vez de regex feito à mão
- Sanitizar o HTML renderizado para que input não confiável não injete scripts
- Fazer debounce ou throttle do parsing para manter a digitação fluida em um documento grande
- Manipular a seleção de um textarea para implementar ações de negrito/itálico/link na barra
- Conectar atalhos de teclado que mapeiam para comandos de formatação
- Persistir um rascunho para que o trabalho não se perca em um reload acidental

## Requisitos Funcionais

1. O editor deve mostrar um painel de origem e um painel de preview ao vivo que atualiza conforme o usuário digita.
2. O markdown deve ser renderizado com um parser de verdade que suporta títulos, listas, links, blocos de código e ênfase.
3. O HTML renderizado deve ser sanitizado antes da inserção para que `<script>` e atributos de manipulador de evento não executem.
4. Uma barra deve aplicar formatação (negrito, itálico, link, código) à seleção atual, envolvendo ou inserindo corretamente.
5. Pelo menos três atalhos de teclado (ex.: negrito, itálico, salvar) devem disparar a ação correspondente.
6. O parsing deve ter debounce ou ser de outra forma limitado para que um documento grande não congele a cada tecla.
7. O documento atual deve persistir localmente e restaurar no reload.

## Marcos Sugeridos

1. **Marco 1 — Parsear e preview:** Conecte o painel de origem a um parser e renderize HTML sanitizado no preview.
2. **Marco 2 — Barra:** Adicione botões de formatação cientes da seleção que envolvem ou inserem sintaxe markdown.
3. **Marco 3 — Atalhos e performance:** Adicione atalhos de teclado e faça debounce do parsing para documentos grandes.
4. **Marco 4 — Persistência:** Auto-salve o rascunho e restaure-o ao carregar.

## Esboço de Dados e Interface

```text
Layout
  ┌───────────────────────────────────────────────┐
  │ Barra: [B] [I] [link] [code] [H1] [lista]      │
  ├──────────────────────┬────────────────────────┤
  │ # Origem (textarea)  │ Preview (sanitizado)    │
  │ **negrito** texto    │ <strong>negrito</strong>│
  │ - item               │ • item                  │
  └──────────────────────┴────────────────────────┘

Estado
  source:   string          // markdown bruto
  html:     string          // derivado: sanitize(parse(source)), com debounce
  draftKey: 'md-editor:draft'

Ação de formatação (pura sobre a seleção)
  applyWrap(source, selStart, selEnd, marker)
    -> { text, newSelStart, newSelEnd }
    // ex.: marker "**" transforma "gato" em "**gato**"

Pipeline
  tecla → debounce(150ms) → parse(md) → sanitize(html) → render
```

## Desafios Extras

- Adicione realce de sintaxe dentro de blocos de código com cerca.
- Sincronize a rolagem dos painéis de origem e preview proporcionalmente.
- Gere um sumário (índice) a partir dos títulos.
- Exporte o documento renderizado para um arquivo HTML autônomo.
- Adicione uma camada de histórico desfazer/refazer acima do nativo do textarea.

## Definição de Pronto

- [ ] Digitar markdown atualiza um preview renderizado corretamente.
- [ ] Um `<script>` ou atributo `onerror` colado é removido e nunca executa.
- [ ] Os botões da barra envolvem a seleção atual em vez de substituí-la ou limpá-la.
- [ ] Digitar em um documento longo permanece responsivo (parsing com debounce/limitado).
- [ ] Um rascunho sobrevive a um reload sem perda de dados.

## Armadilhas Comuns

- Escrever a saída do parser direto em `innerHTML` sem sanitizar — o clássico buraco de XSS armazenado.
- Reparsear de forma síncrona a cada tecla, congelando a aba em um arquivo grande.
- Substituir todo o valor do textarea a cada formatação, apagando o cursor e a pilha de desfazer.
- Fazer seu próprio regex de markdown e se afogar em casos extremos que uma biblioteca já trata.
- Sanitizar a origem em vez do HTML renderizado, quebrando markdown legítimo.

## Recursos

- [MDN: Considerações de segurança do Element.innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML#security_considerations) — por que a injeção de HTML bruto é perigosa.
- [DOMPurify](https://github.com/cure53/DOMPurify) — um sanitizador de HTML testado em batalha.
- [Documentação do marked](https://marked.js.org/) — um parser de markdown rápido e bem mantido.
- [Especificação CommonMark](https://spec.commonmark.org/) — a referência para o que markdown significa.
- [MDN: HTMLTextAreaElement.setSelectionRange()](https://developer.mozilla.org/pt-BR/docs/Web/API/HTMLInputElement/setSelectionRange) — manipulando a seleção para ações da barra.
