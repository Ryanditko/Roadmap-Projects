# Projete um Sistema de Autenticação e Login

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete o sistema de autenticação por trás de uma aplicação web: como usuários se registram, fazem login, permanecem logados entre requisições e saem. A parte sutil é que o HTTP é stateless, então "permanecer logado" exige uma sessão no servidor ou um token assinado. Você vai raciocinar sobre como senhas são armazenadas com segurança, como uma sessão ou token prova a identidade a cada requisição e como se defender de força bruta e credenciais roubadas. Entregue um documento de design cobrindo o fluxo de auth, o modelo de token/sessão e as decisões de segurança — sem código de autenticação funcional.

## Pré-requisitos

- Entendimento de que o HTTP é stateless e cada requisição é independente
- Consciência de que senhas nunca devem ser armazenadas em texto puro
- Familiaridade com cookies e cabeçalhos HTTP em nível básico
- Conforto para raciocinar sobre a diferença entre autenticação e autorização

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um fluxo de registro e login com armazenamento seguro de senhas
- Comparar sessões no servidor com tokens stateless (JWT) e justificar uma escolha
- Raciocinar sobre expiração, refresh e revogação de token
- Projetar defesas contra ataques de força bruta
- Enunciar um trade-off entre escalabilidade stateless e controle de revogação

## Requisitos e Restrições

1. Usuários se registram com um email (ex., `name@example.com`) e senha, e então fazem login.
2. Senhas são armazenadas com um hash lento e com salt — nunca reversível.
3. Uma sessão autenticada persiste entre requisições até expirar ou o usuário sair.
4. O sistema resiste a tentativas de login por força bruta.
5. Logout e revogação de token/sessão devem de fato invalidar o acesso.
6. Estime a escala: 2M usuários, 500K logins/dia — cerca de 6 logins/s em média.

## Abordagem Sugerida

1. Separe a troca única de login da prova de identidade a cada requisição.
2. Projete o armazenamento de senha: um hash lento (bcrypt/argon2) com salt por usuário.
3. Escolha o mecanismo de sessão — store de sessão no servidor vs. token stateless assinado — e note como cada um é validado por requisição.
4. Adicione rate limiting e bloqueio em falhas repetidas.
5. Descreva como o logout funciona no modelo escolhido, incluindo revogação de token.

## Esboço de Arquitetura

```text
Cliente ── POST /register ─> [ Serviço Auth ] ─> [ Store de Usuários (email, pw_hash, salt) ]
Cliente ── POST /login ────> [ Serviço Auth ] ─> verifica hash -> emite token/sessão
Cliente ── GET /me ────────> [ App ] -> valida token/sessão -> [ Store de Sessão? ]
Cliente ── POST /logout ───> [ Serviço Auth ] -> revoga sessão / bloqueia token

API principal
  POST /register  { email, password }   -> 201 { userId }
  POST /login     { email, password }    -> 200 { token }  (+ Set-Cookie)
  GET  /me        (Authorization)         -> 200 { userId } | 401
  POST /logout    (Authorization)         -> 204

Modelo de dados
  users:    user_id (PK) | email | password_hash | salt | created_at
  sessions: session_id (PK) | user_id | expires_at   (se baseado em sessão)
  attempts: email/ip | count | window_start          (para rate limiting)
```

## Tópicos de Aprofundamento

- **Armazenamento de senha:** por que hashes lentos (bcrypt, argon2) superam os rápidos, e o papel de um salt por usuário.
- **Sessões vs. tokens:** o problema de revogação com JWTs stateless e mitigações (expiração curta + refresh, blocklist).
- **Defesa contra força bruta:** rate limiting, bloqueio exponencial e CAPTCHA como escalonamento.

## Entregáveis

- Um diagrama de arquitetura mostrando registro, login, requisição autenticada e logout.
- O contrato da API principal para os endpoints de auth.
- Um modelo de dados para usuários e sessões/tokens.
- Um trade-off descrito: ex., sessões no servidor (revogação fácil, mas um store de sessão compartilhado para escalar) vs. JWTs stateless (escalam livremente, mas revogação é difícil antes da expiração).

## Armadilhas Comuns

- Armazenar senhas com um hash rápido (MD5/SHA-256) ou, pior, em texto puro.
- Usar tokens stateless de vida longa sem caminho de revogação, de modo que um token roubado continua válido.
- Pular o rate limiting, deixando o login aberto a credential stuffing.
- Confundir autenticação (quem você é) com autorização (o que você pode fazer).

## Recursos

- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — orientação prática e confiável.
- [OWASP: Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) — como fazer hash de senhas.
- [RFC 7519: JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519) — o padrão de token.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de sessões, segurança e escalabilidade.
