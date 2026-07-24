# Sistema de UI com Acessibilidade em Primeiro Lugar

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um conjunto de padrões de UI complexos e interativos — diálogos modais, comboboxes, painéis de abas, grids de dados, menus — onde a acessibilidade é a restrição de design primária, não um remendo posterior. Estes são exatamente os widgets que quebram para usuários de teclado e leitor de tela quando construídos de forma ingênua, porque o navegador não lhe dá semântica embutida para eles. Você vai implementá-los seguindo o WAI-ARIA Authoring Practices: roles corretos, foco gerenciado, interação de teclado previsível e anúncios que tornam as mudanças de estado perceptíveis sem visão. A medida de sucesso não é um scan automatizado passando (esses pegam talvez um terço dos problemas), mas um usuário real operando toda a interface apenas com teclado e leitor de tela, sem nunca ficar preso.

## Pré-requisitos

- Habilidades sólidas de HTML e manipulação de DOM
- Entendimento de HTML semântico e da árvore de acessibilidade
- Disposição para testar com um leitor de tela real (NVDA, VoiceOver ou Orca)
- Familiaridade com foco, `tabindex` e tratamento de eventos de teclado

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar os padrões do WAI-ARIA Authoring Practices para widgets complexos corretamente
- Gerenciar o foco de forma deliberada: trapping de foco, restauração e roving tabindex
- Usar roles, estados e live regions ARIA para tornar mudanças dinâmicas perceptíveis
- Testar com navegação apenas por teclado e um leitor de tela real, não só com ferramentas automatizadas
- Atender os critérios de sucesso do WCAG para contraste, visibilidade do foco e tamanho de alvo

## Requisitos Funcionais

1. Todo widget interativo deve ser totalmente operável apenas com o teclado, seguindo as teclas esperadas do seu padrão ARIA.
2. Um diálogo modal deve prender o foco enquanto aberto e restaurar o foco ao gatilho ao fechar.
3. Atualizações dinâmicas (erros de validação, resultados assíncronos) devem ser anunciadas via live regions apropriadas.
4. O foco deve estar sempre visível e nunca ser perdido em um elemento fora da tela ou desanexado.
5. O contraste de cor deve atender ao WCAG AA para texto e componentes de UI.
6. Cada widget deve expor roles e estados corretos na árvore de acessibilidade, verificados no dev tools.
7. A interface deve permanecer utilizável em zoom de 200% e com preferências de movimento reduzido respeitadas.

## Marcos Sugeridos

1. **Marco 1 — Semântica e teclado:** Construa 2–3 widgets com roles corretos e operação completa por teclado.
2. **Marco 2 — Gerenciamento de foco:** Adicione trapping/restauração de foco e roving tabindex onde o padrão exige.
3. **Marco 3 — Anúncios:** Conecte live regions para validação, carregamento e mensagens de resultado assíncrono.
4. **Marco 4 — Verificação:** Teste todo o fluxo com teclado + leitor de tela e corrija o que a automação deixou passar.

## Esboço de Dados e Interface

```text
   Widget: Combobox (padrão WAI-ARIA)
   ┌──────────────────────────────────────────┐
   │ input  role=combobox  aria-expanded=?      │
   │        aria-controls=listbox-id            │
   │        aria-activedescendant=option-id     │
   └───────────────┬────────────────────────────┘
                   ▼ (abre)
   ┌──────────────────────────────────────────┐
   │ ul role=listbox                            │
   │   li role=option  aria-selected=?          │  ← ↑/↓ move, Enter seleciona
   └──────────────────────────────────────────┘

Modelo de foco:  dialog=trap+restaura · menu/grid=roving tabindex
Live regions:    status (polite) p/ resultados · alert (assertive) p/ erros

Metas WCAG (AA):
  contraste de texto   >= 4,5:1  (>= 3:1 texto grande / UI)
  indicador de foco    visível, >= 3:1 contra o fundo
  operável             100% teclado, respeita prefers-reduced-motion
  utilizável em 200%   sem perda de conteúdo ou função
```

## Desafios Extras

- Adicione um skip-link e uma estrutura de landmarks para que usuários de leitor de tela saltem entre regiões.
- Suporte modo de alto contraste / forced-colors sem perder o significado transmitido pela cor.
- Adicione uma checagem automatizada de a11y (axe) na CI como piso, documentando o que ela não pega.
- Forneça uma referência escrita de atalhos de teclado descobrível de dentro da própria UI.

## Definição de Pronto

- [ ] Todo widget é operável apenas com teclado, seguindo os atalhos do seu padrão ARIA.
- [ ] Um leitor de tela anuncia o role, estado e atualizações dinâmicas do widget corretamente.
- [ ] Abrir e fechar um diálogo prende e depois restaura o foco de forma previsível.
- [ ] Todo texto e controles atendem ao contraste WCAG AA, verificado com uma ferramenta de contraste.
- [ ] A interface funciona em zoom de 200% e respeita preferências de movimento reduzido.

## Armadilhas Comuns

- Adicionar roles ARIA a elementos não semânticos esquecendo o comportamento de teclado que o role implica.
- Usar `aria-label` para mascarar uma estrutura acessível ausente em vez de corrigir a semântica.
- Prender o foco em um diálogo mas nunca restaurá-lo, deixando o usuário perdido após o fechamento.
- Abusar de `aria-live="assertive"`, inundando o leitor de tela e abafando alertas importantes.
- Entregar com base apenas em um scan automatizado verde, que ignora a maioria das barreiras reais de usabilidade.

## Recursos

- [WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/) — padrões de referência com o comportamento de teclado esperado.
- [WebAIM: Introduction to Screen Readers](https://webaim.org/articles/screenreader_testing/) — como testar com tecnologia assistiva.
- [MDN: ARIA live regions](https://developer.mozilla.org/pt-BR/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — anunciando mudanças dinâmicas.
- [Referência Rápida do WCAG 2.2](https://www.w3.org/WAI/WCAG22/quickref/) — os critérios de sucesso e como atendê-los.
