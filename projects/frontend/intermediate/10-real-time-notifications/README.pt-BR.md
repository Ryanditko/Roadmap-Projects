# Interface de Notificações em Tempo Real (Toasts)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um sistema de notificações (toasts): mensagens transitórias de diferentes tipos — info, sucesso, aviso, erro — que se empilham, auto-desaparecem após um temporizador e podem ser dispensadas cedo ou carregar um botão de ação. É o tipo de funcionalidade que todo app acaba criando, e é enganosamente traiçoeira. Os temporizadores precisam pausar quando o usuário passa o mouse para que uma mensagem não seja arrancada no meio da leitura, dispensar um toast não pode perturbar a contagem regressiva dos vizinhos e — a parte que a maioria das implementações erra — um leitor de tela precisa ouvir um erro sem que o toast roube o foco. Você vai construí-lo como um pequeno provider/fila em que qualquer componente pode empurrar, mantendo a lógica de temporização fora dos componentes visuais.

## Pré-requisitos

- Conforto com estado de componente, efeitos e limpeza em um framework (React, Vue, Svelte ou Angular)
- Entendimento de temporizadores (`setTimeout`) e da importância de limpá-los
- Familiaridade com um padrão de estado global (context/store) para uma fila cruzando o app
- Consciência de regiões ARIA live e de `role="status"` / `role="alert"`
- Transições CSS básicas e `prefers-reduced-motion`

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma fila de notificações como um provider em que qualquer componente pode despachar
- Gerenciar temporizadores de auto-dispensa por toast, incluindo pausar-ao-passar-o-mouse e retomar
- Distinguir severidades de notificação visual e semanticamente
- Anunciar notificações à tecnologia assistiva sem sequestrar o foco
- Limitar o número de toasts visíveis e tratar o excedente com elegância
- Limpar temporizadores na desmontagem e na dispensa manual para evitar vazamentos

## Requisitos Funcionais

1. Qualquer parte do app deve poder disparar uma notificação via uma função de despacho compartilhada.
2. As notificações devem suportar pelo menos quatro tipos (info, sucesso, aviso, erro) com estilos distintos.
3. Cada toast deve auto-desaparecer após um timeout configurável e ser dispensável cedo via um botão de fechar.
4. Passar o mouse (ou focar) em um toast deve pausar seu temporizador de dispensa; sair deve retomá-lo.
5. Os toasts devem empilhar, limitar a uma contagem máxima visível e enfileirar ou colapsar o excedente.
6. As notificações devem ser anunciadas a leitores de tela — polite para info/sucesso, assertive para erros — sem mover o foco.
7. Um toast pode carregar um botão de ação opcional (ex.: "Desfazer") que executa um callback e dispensa.

## Marcos Sugeridos

1. **Marco 1 — Fila e renderização:** Construa o provider e uma API de despacho; renderize uma pilha de toasts tipados.
2. **Marco 2 — Temporizadores:** Adicione auto-dispensa com pausar-ao-passar-o-mouse, retomar e fechar manual.
3. **Marco 3 — Excedente e ações:** Limite os toasts visíveis, trate o excedente e adicione botões de ação.
4. **Marco 4 — Acessibilidade:** Conecte regiões live e papéis corretos, verifique que não há roubo de foco.

## Esboço de Dados e Interface

```text
Layout (pilha no canto superior direito)
  ┌───────────────────────────────┐
  │ ✓ Salvo com sucesso       [✕] │  ← sucesso, polite
  ├───────────────────────────────┤
  │ ⚠ Sessão expirando        [✕] │  ← aviso
  ├───────────────────────────────┤
  │ ✕ Upload falhou [Repetir] [✕] │  ← erro, assertive, ação
  └───────────────────────────────┘
  (+2 mais…)   ← indicador de excedente quando acima do limite

Estado (provider)
  toasts: Toast[]
  Toast {
    id, type: 'info'|'success'|'warning'|'error',
    message, action?: { label, onClick },
    duration: number, remaining: number, paused: boolean
  }
  MAX_VISIBLE = 4

API de despacho
  notify({ type, message, duration?, action? }) -> id
  dismiss(id)

Acessibilidade
  o contêiner tem uma região aria-live: "polite" (info/sucesso) +
  uma região separada role="alert" para erros; o foco permanece no lugar
```

## Desafios Extras

- Deixe cada toast escolher sua própria posição (canto) e animar entrada/saída a partir daquela borda.
- Agrupe ou remova duplicatas de notificações idênticas em rajada com um selo de contagem.
- Adicione um "centro de notificações" persistente que mantém um histórico dos toasts dispensados.
- Adicione uma barra de progresso sutil mostrando o tempo restante antes da auto-dispensa.
- Suporte toasts baseados em promessa ("carregando → sucesso/erro") para ações assíncronas.

## Definição de Pronto

- [ ] Qualquer componente pode despachar uma notificação através do provider compartilhado.
- [ ] Os toasts auto-desaparecem, pausam ao passar o mouse/focar e retomam ao sair.
- [ ] A contagem visível é limitada e o excedente é tratado sem quebrar o layout.
- [ ] Erros são anunciados de forma assertiva e info de forma polite, com o foco nunca roubado.
- [ ] Os temporizadores são limpos na dispensa e na desmontagem — nenhum callback dispara após a remoção.

## Armadilhas Comuns

- Armazenar os IDs de temporizador de forma solta e nunca limpá-los, disparando dispensa em toasts já removidos.
- Reconstruir todos os temporizadores sempre que a lista muda, resetando toda contagem a cada novo toast.
- Auto-dispensar erros em um temporizador curto antes de o usuário conseguir ler ou agir sobre eles.
- Mover o foco para um toast, prendendo usuários de teclado e interrompendo sua tarefa.
- Depender só da cor para sinalizar severidade, falhando com daltônicos e nas checagens de contraste.

## Recursos

- [MDN: Regiões ARIA live](https://developer.mozilla.org/pt-BR/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — anúncios polite vs assertive.
- [W3C ARIA APG: Padrão Alert](https://www.w3.org/WAI/ARIA/apg/patterns/alert/) — a semântica de uma notificação.
- [MDN: setTimeout / clearTimeout](https://developer.mozilla.org/pt-BR/docs/Web/API/setTimeout) — o ciclo de vida do temporizador a gerenciar.
- [web.dev: prefers-reduced-motion](https://web.dev/articles/prefers-reduced-motion) — animando toasts de forma responsável.
- [Inclusive Components: Notificações/toast](https://inclusive-components.design/notifications/) — um passo a passo de design acessível.
