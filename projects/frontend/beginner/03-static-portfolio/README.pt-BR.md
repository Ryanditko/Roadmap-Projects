# Site de Portfólio Estático

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um site de portfólio pessoal que apresenta quem você é, o que você construiu e como entrar em contato — tudo como páginas estáticas focadas em conteúdo, sem backend. O objetivo não é animação chamativa; é escrever HTML semântico e limpo e um layout CSS responsivo que se leia bem de um celular de 320px a um desktop largo. Você vai estruturar seções reais (introdução, projetos, habilidades, contato), tornar a navegação utilizável por teclado e leitor de tela, e deixar o conteúdo guiar o design em vez do contrário. Bem feito, isso se torna algo que você de fato publica e coloca no currículo.

## Pré-requisitos

- Fundamentos de HTML e a diferença entre elementos semânticos e genéricos
- Modelo de caixa do CSS, Flexbox e o básico de CSS Grid
- Entendimento de media queries e unidades relativas (`rem`, `%`, `vw`)
- Um editor de código e um navegador com ferramentas de desenvolvedor

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Estruturar uma página com marcos semânticos (`header`, `nav`, `main`, `section`, `footer`)
- Construir um layout que se rearranja graciosamente em vários tamanhos de tela sem rolagem horizontal
- Usar Grid e Flexbox para os trabalhos certos (esqueleto da página vs. alinhamento de componente)
- Escrever navegação acessível com ordem lógica de títulos e estados de foco visíveis
- Otimizar imagens com `srcset`/`sizes` responsivos e texto `alt` significativo

## Requisitos Funcionais

1. O site tem seções claramente separadas: introdução/hero, sobre, projetos, habilidades e contato.
2. Uma barra de navegação leva a cada seção e funciona com teclado e toque.
3. O layout é totalmente responsivo, sem estouro horizontal em 320px de largura.
4. Cada entrada de projeto mostra um título, uma descrição curta e um link para o código ou uma demo ao vivo.
5. Todas as imagens têm texto `alt` descritivo; imagens decorativas usam `alt` vazio.
6. Os títulos seguem uma ordem única e lógica (`h1` → `h2` → `h3`) sem pular níveis.
7. O contraste de cores atende ao WCAG AA para texto de corpo e elementos interativos.

## Marcos Sugeridos

1. **Marco 1 — Estrutura e conteúdo:** Escreva o HTML semântico de todas as seções com conteúdo de exemplo real.
2. **Marco 2 — Layout responsivo:** Estilize com Grid/Flexbox e adicione media queries para celular, tablet e desktop.
3. **Marco 3 — Polimento e acessibilidade:** Adicione estados de foco, correções de contraste, imagens responsivas e navegação suave na página.

## Esboço de Dados e Interface

```text
Marcos da página
  header > nav (logo + links de seção)
  main
    section#hero    (nome, cargo, frase de efeito, CTA)
    section#about   (bio curta)
    section#projects (grade de cartões de projeto)
    section#skills  (tags de habilidades agrupadas)
    section#contact (e-mail + links sociais)
  footer (copyright, voltar ao topo)

Cartão de projeto
  title:    string
  summary:  string
  tags:     string[]
  repoUrl / liveUrl: string

Grade desktop          Celular (empilhado)
+----+----+----+        +-----------+
| c1 | c2 | c3 |        |    c1     |
+----+----+----+   -->  +-----------+
| c4 | c5 | c6 |        |    c2     |
+----+----+----+        +-----------+
```

## Desafios Extras

- Adicione um alternador de tema claro/escuro que respeite `prefers-color-scheme`.
- Filtre projetos por tag/categoria sem recarregar a página.
- Adicione animações sutis de revelação ao rolar, condicionadas a `prefers-reduced-motion`.
- Publique em um host estático gratuito e configure um domínio personalizado.

## Definição de Pronto

- [ ] Toda seção é alcançável pela navegação por mouse, toque e teclado.
- [ ] O layout tem zero rolagem horizontal de 320px até larguras de desktop.
- [ ] Todas as imagens de conteúdo têm texto `alt` apropriado.
- [ ] Os níveis de título estão ordenados sem pulos, e há exatamente um `h1`.
- [ ] As cores de texto e interativas passam no contraste WCAG AA.

## Armadilhas Comuns

- Envolver tudo em `<div>`s em vez de marcos semânticos, prejudicando a navegação por leitor de tela.
- Larguras fixas em pixels que forçam rolagem horizontal em telas pequenas.
- Pular níveis de título (`h1` direto para `h4`) por tamanho visual em vez de usar CSS.
- Texto "cinza de designer" de baixo contraste que falha nas verificações de acessibilidade.
- Publicar imagens enormes e não otimizadas que arruínam o tempo de carga no celular.

## Recursos

- [MDN: Referência de elementos HTML](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element) — escolher o elemento semântico certo.
- [web.dev: Learn Responsive Design](https://web.dev/learn/design) — construir layouts que se adaptam.
- [web.dev: Learn Accessibility](https://web.dev/learn/accessibility) — marcos, títulos e contraste.
- [MDN: Imagens responsivas](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Responsive_images) — `srcset` e `sizes`.
