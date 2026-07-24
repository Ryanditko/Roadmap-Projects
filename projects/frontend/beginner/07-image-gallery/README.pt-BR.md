# Galeria de Imagens

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa uma galeria de imagens responsiva: uma grade de miniaturas que abre uma imagem maior em uma sobreposição lightbox ao ser clicada, com navegação anterior/próxima. A grade ensina layout responsivo com CSS Grid; o lightbox ensina algo mais difícil e mais valioso — construir um modal acessível. Um lightbox de verdade deve prender o foco do teclado, fechar no Escape, restaurar o foco ao fechar e ser operável com as setas. Adicione carregamento preguiçoso de imagens e você tem um projeto que toca layout, interação e desempenho em um pacote compacto.

## Pré-requisitos

- Fundamentos de HTML, CSS e JavaScript
- CSS Grid para layouts responsivos
- Tratamento de eventos do DOM, incluindo eventos de teclado
- Uma pasta de imagens de exemplo ou um serviço de imagens de placeholder

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir uma grade responsiva que adapta as colunas à viewport com `auto-fill`/`minmax`
- Implementar um modal/lightbox acessível com prisão e restauração de foco
- Navegar por imagens com botões anterior/próxima e teclas de seta
- Carregar preguiçosamente imagens fora da tela para reduzir a carga inicial
- Fornecer texto `alt` significativo e controles operáveis por teclado

## Requisitos Funcionais

1. As imagens são exibidas em uma grade responsiva que se rearranja pela largura da viewport.
2. Clicar em uma miniatura abre um lightbox mostrando a imagem em tamanho cheio.
3. O lightbox suporta navegação anterior/próxima via botões e teclas de seta.
4. Escape fecha o lightbox e o foco retorna à miniatura que o acionou.
5. Enquanto o lightbox está aberto, o foco do teclado permanece preso dentro dele.
6. Imagens fora da tela carregam preguiçosamente em vez de todas de uma vez no carregamento.
7. Toda imagem tem texto `alt` descritivo, e os controles são rotulados.

## Marcos Sugeridos

1. **Marco 1 — Grade responsiva:** Renderize miniaturas em uma grade que se adapta à largura da tela.
2. **Marco 2 — Lightbox:** Abra uma sobreposição de imagem cheia com controles anterior/próxima.
3. **Marco 3 — Acessibilidade e desempenho:** Adicione prisão de foco, Escape, navegação por teclado e carregamento preguiçoso.

## Esboço de Dados e Interface

```text
Image
  id:      string
  src:     string   (cheia)
  thumb:   string   (pequena)
  alt:     string
  width:   number
  height:  number

Estado do lightbox
  isOpen:  boolean
  index:   number    (imagem atual)

Grade (colunas auto-fill)       Sobreposição do lightbox
+-----+-----+-----+-----+       +------------------------+
| img | img | img | img |       |  [X]                   |
+-----+-----+-----+-----+       |   <   [ imagem ]   >   |
| img | img | img | img |  -->  |                        |
+-----+-----+-----+-----+       |   3 / 12   legenda     |
                                +------------------------+
```

## Desafios Extras

- Adicione gestos de deslizar para dispositivos de toque moverem entre imagens.
- Adicione uma tira de miniaturas dentro do lightbox mostrando a posição no conjunto.
- Adicione um modo de slideshow que avança automaticamente, pausável e respeitando movimento reduzido.
- Use `IntersectionObserver` para carregamento preguiçoso em vez do atributo `loading`.

## Definição de Pronto

- [ ] A grade se rearranja sem estouro do celular ao desktop.
- [ ] O lightbox abre, navega e fecha por mouse e por teclado.
- [ ] O foco fica preso enquanto aberto e é restaurado ao acionador ao fechar.
- [ ] Imagens abaixo da dobra não carregam até serem necessárias.
- [ ] Todas as imagens carregam texto `alt` apropriado.

## Armadilhas Comuns

- Construir o lightbox como uma sobreposição simples que deixa o foco e o Tab escaparem para trás dele.
- Não restaurar o foco ao fechar, fazendo usuários de teclado perderem o lugar.
- Esquecer de definir dimensões explícitas de imagem, causando deslocamento de layout enquanto elas carregam.
- Carregar imagens em resolução cheia como miniaturas, desperdiçando banda.
- Fazer os controles de fechar e navegar serem `<div>`s não focáveis.

## Recursos

- [MDN: CSS Grid Layout](https://developer.mozilla.org/pt-BR/docs/Web/CSS/CSS_grid_layout) — grades responsivas com `minmax` e `auto-fill`.
- [MDN: Lazy loading](https://developer.mozilla.org/pt-BR/docs/Web/Performance/Lazy_loading) — adiar imagens fora da tela.
- [WAI-ARIA: Dialog (Modal) pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/) — prisão de foco e comportamento de teclado.
- [web.dev: prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion) — respeitar preferências de movimento.
