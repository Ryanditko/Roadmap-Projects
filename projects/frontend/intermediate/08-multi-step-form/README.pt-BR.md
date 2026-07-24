# Formulário Multietapas (Wizard)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um formulário multietapas — um wizard — que divide um formulário longo (digamos, conta → perfil → pagamento → revisão) em várias telas com um indicador de progresso, validação por etapa e uma revisão final antes de enviar. Um único formulário gigante é fácil; o wizard é mais difícil porque o estado agora precisa sobreviver à navegação entre etapas, uma etapa não pode deixar o usuário avançar até que seus campos sejam válidos, algumas etapas podem ser condicionais a respostas anteriores, e tudo precisa permanecer acessível e recuperável se o usuário der reload no meio do fluxo. Este projeto é, na verdade, sobre arquitetura de estado de formulário e o momento da validação, não sobre markup.

## Pré-requisitos

- Conforto com inputs de formulário controlados e estado de componente em um framework (React, Vue, Svelte ou Angular)
- Entendimento de validação (em nível de campo e de etapa) e de quando executá-la
- Familiaridade com estado derivado (progresso, "pode avançar?") a partir de um único modelo de formulário
- Conhecimento básico de `localStorage` ou persistência de sessão
- Opcional: uma biblioteca de formulário/validação (React Hook Form, Formik, Zod, Yup) versus feito à mão

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Manter todo o formulário em um único modelo de estado que persiste entre a navegação de etapas
- Validar uma etapa antes de permitir a navegação para frente, permitindo movimento livre para trás
- Modelar etapas condicionais que aparecem ou somem com base em respostas anteriores
- Construir um indicador de progresso acessível e gerenciar o foco na troca de etapa
- Montar uma etapa de revisão que relê todos os dados coletados antes do envio
- Persistir e restaurar o progresso para que um reload não perca os dados inseridos

## Requisitos Funcionais

1. O formulário deve abranger pelo menos três etapas com um indicador de progresso visível mostrando a etapa atual e o total.
2. Cada etapa deve validar seus próprios campos, e "Próximo" deve ser bloqueado até que a etapa atual seja válida.
3. "Voltar" deve retornar à etapa anterior com todos os valores previamente inseridos intactos.
4. Ao menos uma etapa deve ser condicional — exibida ou pulada com base em um valor de uma etapa anterior.
5. Uma etapa final de revisão deve exibir todos os dados inseridos e permitir saltar de volta para editar qualquer etapa.
6. No envio, o payload montado deve ser validado como um todo antes de a requisição disparar.
7. O progresso do formulário deve persistir para que um reload restaure a etapa atual e os valores inseridos.

## Marcos Sugeridos

1. **Marco 1 — Etapas e navegação:** Renderize etapas com próximo/voltar e um indicador de progresso sobre um modelo de estado.
2. **Marco 2 — Portão de validação:** Adicione validação por etapa que barra a navegação para frente e mostra erros.
3. **Marco 3 — Condicional e revisão:** Adicione uma etapa condicional e uma tela de revisão com links de edição.
4. **Marco 4 — Persistir e enviar:** Salve o progresso, restaure no reload e valide o payload completo no envio.

## Esboço de Dados e Interface

```text
Layout
  ①──②──③──④    ← progresso: etapa 2 de 4 ativa
  ┌───────────────────────────────┐
  │ Etapa 2: Perfil                │
  │ [ Nome ...... ]  (texto erro)  │
  │ [ Bio  ...... ]                │
  ├───────────────────────────────┤
  │       [ Voltar ]  [ Próximo → ]│  ← Próximo desabilitado até válido
  └───────────────────────────────┘

Estado (modelo único)
  currentStep: number
  data: { account: {...}, profile: {...}, payment?: {...} }
  errors: Record<field, string>
  visitedSteps: Set<number>

Definição de etapa
  steps: {
    id, title,
    fields: string[],
    validate(data) -> Record<field,string>,   // vazio = válido
    isVisible(data) -> boolean                 // etapas condicionais
  }[]

Persistência
  salve data + currentStep no sessionStorage a cada mudança; restaure ao carregar
```

## Desafios Extras

- Adicione um link "salvar e continuar depois" que retoma a partir de um token armazenado.
- Anime as transições de etapa respeitando `prefers-reduced-motion`.
- Mostre uma barra lateral de resumo que atualiza ao vivo conforme o usuário preenche cada etapa.
- Suporte deep-linking para uma etapa via URL, protegido pela validação das etapas anteriores.
- Adicione validação assíncrona para um campo (ex.: disponibilidade de nome de usuário).

## Definição de Pronto

- [ ] Dados inseridos em uma etapa continuam presentes após navegar para longe e voltar.
- [ ] "Próximo" é bloqueado com erros visíveis até que a etapa atual seja válida.
- [ ] Uma etapa condicional aparece ou é pulada corretamente com base no input anterior.
- [ ] A etapa de revisão reflete todos os dados e liga de volta para editar cada etapa.
- [ ] Um reload restaura a etapa atual e todos os valores inseridos.

## Armadilhas Comuns

- Armazenar os dados de cada etapa em um componente separado que é desmontado, perdendo valores na navegação.
- Validar só no envio, de modo que o usuário chega à última etapa antes de descobrir que a etapa 1 estava errada.
- Deixar dados obsoletos de uma etapa condicional oculta vazarem para o payload final.
- Não mover o foco para o título da nova etapa, deixando usuários de teclado e leitor de tela perdidos.
- Bloquear a navegação para trás atrás da validação — voltar deve ser sempre livre.

## Recursos

- [MDN: Validação de formulário no cliente](https://developer.mozilla.org/pt-BR/docs/Learn/Forms/Form_validation) — fundamentos de validação.
- [web.dev: Boas práticas de formulário de login](https://web.dev/articles/sign-in-form-best-practices) — UX de formulário com múltiplos campos e autofill.
- [W3C ARIA APG: Gerenciamento de foco](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/) — movendo o foco na troca de etapa.
- [Documentação do React Hook Form](https://react-hook-form.com/) — uma biblioteca de estado/validação de formulário amplamente usada.
- [MDN: Window.sessionStorage](https://developer.mozilla.org/pt-BR/docs/Web/API/Window/sessionStorage) — persistindo o progresso do wizard.
