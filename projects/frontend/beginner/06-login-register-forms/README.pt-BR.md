# Formulários de Login e Cadastro

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa o front-end da autenticação: um formulário de login e um de cadastro com validação real no cliente. Não há backend — você simula o envio — então a lição é inteiramente sobre a UX de formulários, onde reside boa parte do ofício de frontend. Você vai validar campos conforme o usuário digita, mostrar mensagens de erro precisas ao lado dos campos certos, desabilitar o envio até que o formulário seja válido, e fazer tudo isso de forma acessível para que usuários de leitor de tela saibam exatamente o que deu errado. Acertar o momento e a mensagem da validação é a diferença entre um formulário que as pessoas completam e um que abandonam.

## Pré-requisitos

- Formulários HTML, inputs e o elemento `<label>`
- Tratamento de eventos em JavaScript (`input`, `submit`, `blur`)
- Expressões regulares ou validação nativa para verificações de padrão
- Uma noção básica de por que a validação no cliente é UX, não segurança

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Validar campos nos eventos certos (ao sair / ao enviar, não agressivamente à primeira tecla)
- Associar mensagens de erro a inputs usando `aria-describedby` e `aria-invalid`
- Gerenciar o estado do formulário: valores, campos tocados, erros e lógica de envio desabilitado
- Construir um indicador de força de senha e um alternador mostrar/ocultar senha
- Prevenir envio duplo e dar feedback de sucesso claro

## Requisitos Funcionais

1. O formulário de cadastro valida formato de e-mail, regras de senha e correspondência da confirmação de senha.
2. Erros aparecem ao lado do campo relevante e são anunciados à tecnologia assistiva.
3. O botão de envio é desabilitado (ou bloqueia o envio) enquanto o formulário é inválido.
4. Um indicador de força de senha atualiza conforme o usuário digita.
5. Um alternador mostrar/ocultar revela ou mascara o campo de senha.
6. O formulário de login valida campos obrigatórios e mostra um único erro de nível de formulário em caso de "falha".
7. Enviar mostra um estado de carregamento, depois uma confirmação de sucesso, prevenindo envios duplos.

## Marcos Sugeridos

1. **Marco 1 — Estrutura e rótulos:** Construa ambos os formulários com inputs devidamente rotulados e acessíveis.
2. **Marco 2 — Validação:** Adicione validação de campo, mensagens de erro e bloqueio de envio.
3. **Marco 3 — Feedback e polimento:** Adicione medidor de força, mostrar/ocultar, carregamento e estados de sucesso.

## Esboço de Dados e Interface

```text
Estado do campo (por input)
  value:   string
  touched: boolean
  error:   string | null

Campos do cadastro
  email             -> padrão de e-mail válido
  password          -> tamanho mínimo, regras de caracteres mistos
  confirmPassword   -> deve ser igual a password
  acceptTerms       -> deve estar marcado

Layout (cadastro)
+----------------------------------+
| Email    [ .................. ]  |
|          texto de erro (aria)    |
| Senha    [ ............ ] [👁]    |
|          [====----] Média        |
| Confirme [ .................. ]  |
| [x] Aceito os termos             |
| [        Criar conta          ]  |  <- desabilitado até ser válido
+----------------------------------+
```

## Desafios Extras

- Adicione um formulário de "esqueci minha senha" com sua própria validação.
- Persista um rascunho do formulário de cadastro no `sessionStorage`.
- Adicione verificação em linha de "disponível/em uso" de nome de usuário contra uma lista simulada.
- Suporte envio ao Enter e navegação completa por teclado com uma ordem lógica de tabulação.

## Definição de Pronto

- [ ] Todo input tem um rótulo associado programaticamente.
- [ ] Erros estão vinculados aos inputs via `aria-describedby` e definem `aria-invalid`.
- [ ] O envio é bloqueado enquanto qualquer campo é inválido.
- [ ] O alternador de senha muda o tipo do input sem perder o valor.
- [ ] Um envio não pode disparar duas vezes, e o sucesso é claramente comunicado.

## Armadilhas Comuns

- Validar a cada tecla desde o primeiro caractere, fazendo os usuários verem erros antes de terminarem de digitar.
- Usar `placeholder` como substituto de `<label>`, que desaparece e falha na acessibilidade.
- Mostrar uma mensagem genérica de "formulário inválido" em vez de erros específicos por campo.
- Tratar a validação no cliente como segurança — a verificação real deve ocorrer em um servidor.
- Esquecer de focar ou anunciar erros, deixando usuários de leitor de tela travados.

## Recursos

- [MDN: Validação de formulário no cliente](https://developer.mozilla.org/pt-BR/docs/Learn/Forms/Form_validation) — restrições e mensagens personalizadas.
- [web.dev: Sign-in form best practices](https://web.dev/articles/sign-in-form-best-practices) — orientação de UX de formulários do mundo real.
- [MDN: aria-invalid](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-invalid) — sinalizar campos inválidos à tecnologia assistiva.
- [WAI: Instruções e erros de formulário](https://www.w3.org/WAI/tutorials/forms/) — padrões de formulários acessíveis.
