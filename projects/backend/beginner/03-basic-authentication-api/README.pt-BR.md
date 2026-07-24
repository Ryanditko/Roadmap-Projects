# API de Autenticação Básica

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 5–8 horas

## Visão Geral

Construa a camada de login que fica na frente de uma API protegida: um usuário envia credenciais, o servidor as verifica e emite um token, e as requisições seguintes devem carregar esse token para alcançar rotas protegidas. Você parte de um pequeno conjunto de usuários predefinidos para focar no fluxo em si — autenticação versus autorização, emissão de token e o middleware que faz o controle de acesso — em vez de cadastro de usuários ou banco de dados. Todo backend real tem essa camada; aqui você a constrói a partir dos princípios.

## Pré-requisitos

- Base sólida em construir endpoints HTTP ([API REST Simples de Tarefas](../01-simple-rest-api-task-management/) recomendada antes)
- Entendimento de cabeçalhos HTTP, especialmente o `Authorization`
- Um framework web que suporte middleware/hooks
- Consciência de que credenciais nunca devem ser logadas ou fixadas em texto puro

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Distinguir autenticação (quem você é) de autorização (o que você pode fazer)
- Verificar uma senha contra um hash armazenado em vez de comparar texto puro
- Emitir um token no login e validá-lo nas requisições protegidas
- Implementar middleware que protege rotas com base na presença e validade do token
- Raciocinar sobre expiração de token e por que sessões não devem durar para sempre
- Retornar os códigos de status certos para falhas de auth: 401 versus 403

## Requisitos Funcionais

1. O sistema deve expor um endpoint de login que aceita um usuário e uma senha.
2. O sistema deve verificar a senha contra um hash armazenado, nunca uma comparação de texto puro.
3. Em caso de sucesso, o sistema deve retornar um token; em caso de falha, deve retornar 401 sem detalhar qual campo estava errado.
4. Endpoints protegidos devem rejeitar requisições sem um token válido com 401.
5. O sistema deve expor ao menos um endpoint protegido que retorna a identidade de quem chamou.
6. Tokens devem carregar ou referenciar uma expiração, após a qual são rejeitados.
7. O sistema deve suportar logout ou invalidação de token.

## Marcos Sugeridos

1. **Marco 1 — Verificar credenciais:** Armazene usuários com senhas em hash e retorne um token em um login correto.
2. **Marco 2 — Proteger rotas:** Adicione middleware que lê o cabeçalho `Authorization` e bloqueia tokens inválidos.
3. **Marco 3 — Ciclo de vida e papéis:** Adicione expiração, logout/invalidação e opcionalmente uma checagem de papel (admin vs usuário).

## Esboço de Dados e Interface

```text
User (dados iniciais)
  username:     string
  passwordHash: string   (nunca texto puro)
  role:         "admin" | "user"

POST /login              body: { username, password }
                         -> 200 { token, expiresAt } | 401
GET  /me                 header: Authorization: Bearer <token>
                         -> 200 { username, role } | 401
POST /logout             -> 204   (invalida o token)

Regra da requisição protegida:
  token ausente/expirado/inválido -> 401
  token válido mas papel insuficiente -> 403
```

## Desafios Extras

- Substitua tokens opacos por JWTs assinados e verifique a assinatura a cada requisição.
- Adicione autorização por papel para que rotas exclusivas de admin rejeitem usuários comuns com 403.
- Adicione um fluxo de refresh-token para renovar o acesso sem reinserir credenciais.
- Aplique rate-limit em logins falhos para desacelerar tentativas de força bruta.

## Definição de Pronto

- [ ] O login só tem sucesso com credenciais corretas e retorna um token.
- [ ] Senhas são armazenadas e comparadas como hashes, verificável lendo o código.
- [ ] Rotas protegidas retornam 401 para tokens ausentes, malformados ou expirados.
- [ ] Senha errada e usuário desconhecido retornam o mesmo 401 genérico.
- [ ] Logout (ou expiração) faz um token antes válido parar de funcionar.

## Armadilhas Comuns

- Comparar senhas com `==` em vez de uma função de verificação de hash — isso vaza e é inseguro.
- Dizer ao cliente se o usuário ou a senha estava errado, o que ajuda atacantes a enumerar usuários.
- Confundir 401 (não autenticado) com 403 (autenticado, mas não permitido).
- Assinar tokens com um segredo fixo commitado no repositório; leia-o da configuração.

## Recursos

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — os "faça" e "não faça" de referência.
- [MDN: Cabeçalho HTTP Authorization](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Authorization) — como o token viaja.
- [Introdução ao jwt.io](https://jwt.io/introduction) — o que é um JSON Web Token e como é estruturado.
- [MDN: 401 vs 403](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status/401) — quando usar cada código de status de auth.
