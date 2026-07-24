# Arquitetura de Microfrontends

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Divida um único frontend grande em várias aplicações construídas e implantadas de forma independente que se compõem em um produto único e coeso em tempo de execução — o padrão por trás de UIs em larga escala no Spotify, na IKEA e na DAZN. Um "shell" (host) carrega apps de funcionalidades (remotes) sob demanda, para que os times entreguem no seu próprio ritmo sem um trem de release compartilhado. As partes difíceis não são os mecanismos de carregamento, mas as fronteiras: versões de dependências compartilhadas, comunicação entre apps, roteamento consistente e um shell que degrada com elegância quando um remote falha. Este projeto força você a encarar o trade-off no coração de todo sistema de microfrontends — autonomia dos times versus consistência do produto — e a defendê-lo com medições reais, não com opinião.

## Pré-requisitos

- Uma SPA de nível de produção já construída (o [Painel Administrativo](../../intermediate/05-admin-panel/) é uma boa base)
- Domínio sólido de módulos ES, empacotamento e code splitting
- Conforto com roteamento no cliente e seu modelo de histórico
- Familiaridade com um bundler que suporte federação (Vite, Webpack 5 ou Rspack)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Compor múltiplos apps implantados de forma independente atrás de um shell em tempo de execução
- Compartilhar bibliotecas pesadas (runtime do framework, design system) como singletons para evitar duplicação
- Projetar um contrato de comunicação desacoplado entre remotes sem estado global compartilhado
- Coordenar o roteamento para que deep links resolvam corretamente através das fronteiras dos apps
- Isolar falhas para que um remote quebrado nunca apague a página inteira

## Requisitos Funcionais

1. Um shell host deve carregar dinamicamente pelo menos duas aplicações remote construídas de forma independente em tempo de execução.
2. Cada remote deve ser construível e implantável por conta própria, sem reconstruir o shell.
3. As bibliotecas de framework e design system compartilhadas devem resolver para uma única instância, não uma cópia por remote.
4. O shell deve ser dono do roteamento de nível superior e delegar sub-rotas ao remote responsável.
5. Os remotes devem se comunicar por um contrato explícito (eventos customizados ou um event bus injetado), nunca acessando as entranhas um do outro.
6. Se um remote falhar ao carregar ou lançar erro na montagem, o shell deve renderizar um fallback e manter o resto da página utilizável.
7. Os metadados de versão de cada remote carregado devem ser observáveis em tempo de execução (ex.: logados ou exibidos em um painel de debug).

## Marcos Sugeridos

1. **Marco 1 — Shell + um remote:** Suba um host que carrega um único remote via module federation e o renderiza em uma rota.
2. **Marco 2 — Segundo remote e deps compartilhadas:** Adicione um segundo remote e configure singletons compartilhados; prove que só uma cópia do framework é enviada.
3. **Marco 3 — Comunicação e roteamento:** Conecte a mensageria entre remotes e o roteamento por deep link de ponta a ponta entre fronteiras.
4. **Marco 4 — Resiliência e versionamento:** Adicione error boundaries, fallbacks de carga e relato de versão em tempo de execução por remote.

## Esboço de Dados e Interface

```text
                 ┌──────────────────────────────┐
                 │          Shell (host)         │
                 │  roteamento · layout · auth   │
                 │  singletons compartilhados ↓  │
                 └───────┬───────────┬───────────┘
        carrega em runtime │           │ carrega em runtime
                 ┌─────────▼──┐    ┌────▼───────┐
                 │  Remote A  │    │  Remote B  │
                 │ (repo e    │    │ (repo e    │
                 │  build     │    │  build     │
                 │  próprios) │    │  próprios) │
                 └─────┬──────┘    └─────┬──────┘
                       └──── event bus ──┘   (contrato desacoplado)

Singletons compartilhados: runtime do framework, design system, i18n
Comunicação:               window CustomEvent | pub/sub injetado | estado na URL
Modo de falha:             carga do remote rejeita -> shell renderiza <Fallback/>

Metas não funcionais:
  JS só do shell   <= 100 KB gzipado
  entry do remote  <= 30 KB gzipado
  remote quebrado  -> resto da página segue interativo
```

## Desafios Extras

- Adicione um registro em tempo de execução para que remotes possam ser adicionados sem editar a config do shell.
- Implemente CI/CD independente onde cada remote publica um `remoteEntry` versionado em uma CDN.
- Suporte dois frameworks em remotes diferentes (ex.: React + Vue) para provar isolamento real.
- Adicione composição no servidor ou um prerender de app-shell para desempenho de primeira pintura.

## Definição de Pronto

- [ ] Dois remotes implantam de forma independente e o shell adota novas versões sem um rebuild.
- [ ] A análise de bundle prova que as bibliotecas compartilhadas carregam uma vez, não uma por remote.
- [ ] Um remote deliberadamente quebrado mostra um fallback enquanto o resto da página segue interativo.
- [ ] Deep links para a sub-rota de um remote carregam corretamente em um carregamento frio.
- [ ] Os remotes trocam pelo menos uma mensagem pelo contrato acordado, sem imports diretos entre eles.

## Armadilhas Comuns

- Versões incompatíveis de dependências compartilhadas fazendo duas cópias do framework carregarem e os hooks quebrarem de forma sutil.
- Acoplar remotes por um objeto global compartilhado em vez de um contrato explícito — recria o monólito do qual você fugia.
- Deixar cada remote dono do roteamento, produzindo escritas de histórico conflitantes e um botão voltar quebrado.
- Ignorar o caminho de falha, de modo que um 404 em um `remoteEntry` apaga a tela inteira.
- Duplicar resets de CSS e design tokens por remote, causando desvio visual entre fronteiras.

## Recursos

- [Documentação do Module Federation](https://module-federation.io/) — o guia canônico do runtime de federação e dos escopos compartilhados.
- [martinfowler.com: Micro Frontends](https://martinfowler.com/articles/micro-frontends.html) — o artigo de referência sobre a arquitetura e seus trade-offs.
- [web.dev: Reduza payloads de JavaScript com code splitting](https://web.dev/articles/reduce-javascript-payloads-with-code-splitting) — a base de empacotamento sobre a qual a federação se apoia.
- [MDN: CustomEvent](https://developer.mozilla.org/pt-BR/docs/Web/API/CustomEvent) — uma forma agnóstica de framework para construir um event bus desacoplado.
