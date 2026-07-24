# Alternador de Tema (Claro / Escuro / Sistema)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um sistema de temas que permite ao usuário alternar entre os modos claro, escuro e "seguir o sistema", com a escolha lembrada entre visitas. A demonstração parece trivial — trocar algumas cores — mas fazê-la bem toca em problemas mais sutis: alimentar toda a UI a partir de propriedades customizadas CSS para que a troca de um único atributo re-tematize tudo, respeitar a preferência do SO via `prefers-color-scheme` e eliminar o "flash do tema errado" que acontece quando a página pinta antes de o seu script rodar. Você também vai garantir que ambas as paletas realmente passem nos requisitos de contraste, porque um tema escuro que falha no WCAG é só um tipo diferente de quebrado.

## Pré-requisitos

- Fundamentos sólidos de CSS, especialmente propriedades customizadas (variáveis) e seletores
- Conforto com scripting básico de DOM (ler/definir atributos no `<html>`)
- Entendimento de `localStorage` para persistência
- Familiaridade com media queries, especificamente `prefers-color-scheme`
- Consciência de contraste de cor e das razões de contraste do WCAG

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Arquitetar um tema com propriedades customizadas CSS para que um atributo controle todas as cores
- Suportar três modos — claro, escuro e sistema — com o sistema acompanhando o SO ao vivo
- Persistir a escolha explícita do usuário e recorrer à preferência do sistema caso contrário
- Prevenir o flash de tema incorreto (FOUC) com um script inline antes da pintura
- Verificar que ambas as paletas atendem ao contraste WCAG AA para texto e elementos interativos
- Respeitar `prefers-reduced-motion` ao transicionar entre temas

## Requisitos Funcionais

1. A UI deve ser tematizável inteiramente por propriedades customizadas CSS ligadas a um atributo `data-theme` (ou `class`) na raiz.
2. Um controle deve alternar entre os modos claro, escuro e sistema.
3. No modo sistema, o tema deve seguir a configuração do SO e atualizar ao vivo se a preferência do SO mudar.
4. Uma escolha explícita do usuário deve persistir entre reloads; sem escolha, o app usa o padrão sistema.
5. O tema correto deve ser aplicado antes da primeira pintura, sem flash visível das cores erradas.
6. Ambas as paletas, clara e escura, devem atender ao contraste WCAG AA para texto de corpo e controles.
7. As transições de tema devem ser desabilitadas ou reduzidas quando `prefers-reduced-motion: reduce` estiver ativo.

## Marcos Sugeridos

1. **Marco 1 — Paleta de tokens:** Defina as paletas clara e escura como propriedades customizadas CSS alternadas por um atributo raiz.
2. **Marco 2 — Alternador:** Adicione um controle para definir claro/escuro/sistema e aplique-o à raiz.
3. **Marco 3 — Persistir e sistema:** Persista a escolha, use sistema como padrão e reaja a mudanças ao vivo do SO.
4. **Marco 4 — Polimento:** Elimine o FOUC com um script antes da pintura e verifique contraste e movimento reduzido.

## Esboço de Dados e Interface

```text
O atributo da raiz controla tudo
  <html data-theme="dark">
  :root                 { --bg: #fff; --fg: #111; --accent: #2563eb; }
  [data-theme="dark"]   { --bg: #111; --fg: #eee; --accent: #60a5fa; }
  body { background: var(--bg); color: var(--fg); }

Lógica de resolução
  choice: 'light' | 'dark' | 'system'      (persistido em localStorage)
  system: matchMedia('(prefers-color-scheme: dark)').matches
  efetivo = choice === 'system' ? (system ? 'dark' : 'light') : choice

Atualizações ao vivo do SO
  matchMedia('(prefers-color-scheme: dark)')
    .addEventListener('change', reaplicarSeModoSistema)

Antes da pintura (evitar FOUC): um script inline no <head> define data-theme
  a partir do localStorage ANTES de o body dirigido pela folha de estilo renderizar
```

## Desafios Extras

- Adicione mais temas (ex.: alto contraste, sépia) selecionáveis por um dropdown.
- Adicione uma troca automática agendada (escuro após o pôr do sol) usando a hora local.
- Ofereça um seletor de cor de destaque que escreve uma propriedade customizada.
- Exporte/importe uma configuração de tema como JSON.
- Anime a alternância com uma view transition, condicionada ao movimento reduzido.

## Definição de Pronto

- [ ] Toda cor vem de uma propriedade customizada; trocar o atributo da raiz re-tematiza todo o app.
- [ ] Os modos claro, escuro e sistema funcionam, e o sistema acompanha uma mudança ao vivo do SO.
- [ ] O modo escolhido persiste entre reloads e usa sistema como padrão quando não definido.
- [ ] Não há flash do tema errado na primeira pintura.
- [ ] Ambas as paletas passam no contraste WCAG AA e as transições respeitam o movimento reduzido.

## Armadilhas Comuns

- Fixar cores nos componentes em vez de referenciar tokens, fazendo metade da UI ignorar o tema.
- Ler o `localStorage` em um bundle de execução tardia, causando um flash visível antes de o tema aplicar.
- Tratar "sistema" como uma leitura única em vez de assinar os eventos de mudança do `matchMedia`.
- Publicar uma paleta escura que parece bonita mas falha no contraste do texto secundário.
- Aplicar um `transition: all` global que faz a página inteira sacudir a cada troca de tema.

## Recursos

- [MDN: Usando propriedades customizadas CSS](https://developer.mozilla.org/pt-BR/docs/Web/CSS/Using_CSS_custom_properties) — a base da tematização por tokens.
- [MDN: prefers-color-scheme](https://developer.mozilla.org/pt-BR/docs/Web/CSS/@media/prefers-color-scheme) — detectando o tema do SO.
- [web.dev: prefers-color-scheme](https://web.dev/articles/prefers-color-scheme) — construindo um modo escuro robusto, incluindo o FOUC.
- [WebAIM: Contraste e acessibilidade de cor](https://webaim.org/articles/contrast/) — atendendo às razões de contraste do WCAG.
- [MDN: prefers-reduced-motion](https://developer.mozilla.org/pt-BR/docs/Web/CSS/@media/prefers-reduced-motion) — honrando as preferências de movimento.
