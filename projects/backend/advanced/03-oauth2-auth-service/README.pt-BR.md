# Serviço de Autenticação de Alta Escala (OAuth2)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa o serviço em que todos os outros serviços confiam: um servidor de autorização centralizado que emite e valida tokens OAuth2 para muitas aplicações cliente de uma vez. Você implementará os fluxos que alimentam os botões "Entrar com…" e o acesso máquina-a-máquina, e depois os endurecerá para escala — onde a validação de token acontece milhões de vezes por minuto e uma chave de assinatura vazada é um incidente que afeta a empresa inteira. O trabalho interessante não é o caminho feliz, e sim o ciclo de vida: rotacionar chaves sem invalidar sessões vivas, revogar um refresh token roubado instantaneamente e permitir que servidores de recurso validem access tokens sem te chamar a cada requisição.

## Pré-requisitos

- Domínio sólido de HTTP, TLS e de como cookies e cabeçalhos `Authorization` trafegam
- Conforto com criptografia de chave pública (assinatura vs. encriptação, pares de chaves)
- Experiência construindo APIs REST com persistência e uma camada de cache
- Familiaridade com JSON Web Tokens e suas claims; leia a spec OAuth2 antes de começar
- Uma stack de backend de sua escolha (Node, Go, Java, Python) além de um datastore e um cache tipo Redis

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar o fluxo authorization code com PKCE e o fluxo client credentials
- Raciocinar sobre tempos de vida de access token vs. refresh token e sua rotação
- Assinar tokens com chaves rotativas expostas por um endpoint JWKS
- Projetar validação stateless para que servidores de recurso escalem de forma independente
- Revogar e introspeccionar tokens, e auditar toda ação sensível
- Ponderar o trade-off entre JWTs autocontidos e tokens de referência opacos

## Requisitos Funcionais

1. O serviço deve implementar o grant authorization code com PKCE e o grant client credentials conforme a RFC 6749 e a RFC 7636.
2. Deve emitir access tokens de curta duração e refresh tokens de duração maior, com rotação de refresh token invalidando o token anterior no uso.
3. Deve assinar tokens com uma chave assimétrica e publicar chaves públicas em um endpoint JWKS para que servidores de recurso validem offline.
4. Deve suportar autorização baseada em escopo e rejeitar requisições por escopos para os quais um cliente não está registrado.
5. Deve expor endpoints de introspecção de token (RFC 7662) e de revogação (RFC 7009).
6. Deve detectar e rejeitar reuso de refresh token, tratando-o como sinal de comprometimento e revogando a família de tokens.
7. **Disponibilidade:** a validação não deve depender de o auth server estar acessível; o JWKS é cacheado com um TTL.
8. **Vazão (throughput):** os endpoints de token e introspecção devem sustentar altas taxas de requisição; caminhos quentes (busca de cliente, material de chave) devem ser cacheados.
9. **Consistência:** a revogação deve se propagar aos servidores de recurso dentro de uma janela de defasagem limitada e documentada.

## Marcos Sugeridos

1. **Marco 1 — Authorization code + PKCE:** Registre clientes, rode o fluxo completo de redirect, emita um access token assinado.
2. **Marco 2 — Refresh e rotação:** Adicione refresh tokens com rotação e detecção de reuso.
3. **Marco 3 — Chaves e validação:** Publique o JWKS, rotacione chaves com sobreposição, valide tokens offline.
4. **Marco 4 — Introspecção, revogação e auditoria:** Adicione os endpoints de gerenciamento e um log de auditoria append-only.

## Esboço de Dados e Interface

```text
Visão de componentes

  [App Cliente] --redirect--> [Auth Server] --consentimento--> [Usuário]
        |  code + verifier PKCE        |
        v                              v
   POST /token  <----------------  [Client Store]
        |                              |
   access + refresh              [Chaves de Assinatura] --pública--> GET /.well-known/jwks.json
        |
        v
  [Servidor de Recurso] --valida offline via JWKS cacheado--> permite/nega
        ^
        |-- introspect (tokens opacos) / consulta lista de revogação

Endpoints
  GET  /authorize        (response_type=code, code_challenge, scope, state)
  POST /token            (grant_type: authorization_code | refresh_token | client_credentials)
  POST /introspect       -> { active, scope, client_id, exp }
  POST /revoke           -> 200
  GET  /.well-known/jwks.json
```

## Desafios Extras

- Adicione o device authorization grant (RFC 8628) para TVs e ferramentas de CLI.
- Suporte OpenID Connect: adicione um `id_token` e um endpoint `/userinfo`.
- Adicione autenticação multifator como um step-up durante o fluxo de autorização.
- Implemente registro dinâmico de clientes e rate limiting por cliente.

## Definição de Pronto

- [ ] Um cliente completa o fluxo code + PKCE e recebe um access token funcional e com escopo correto.
- [ ] Access tokens validam offline contra o JWKS sem nenhuma chamada ao auth server.
- [ ] A rotação de chaves mantém tokens existentes válidos até expirarem (janela de chaves sobrepostas).
- [ ] Reusar um refresh token rotacionado revoga a família inteira de tokens e é registrado.
- [ ] Os endpoints de revogação e introspecção se comportam conforme a RFC, e toda emissão/revogação é auditada.

## Armadilhas Comuns

- Access tokens de longa duração: se não podem ser revogados, mantenha-os curtos e apoie-se na rotação de refresh.
- Pular `state` e PKCE, deixando o fluxo aberto a CSRF e interceptação do authorization code.
- Rotacionar chaves de assinatura sem janela de sobreposição, invalidando instantaneamente todo token vivo.
- Colocar dados sensíveis ou que mudam rápido em um JWT — você não pode "desemitir" antes de expirar.
- Armazenar refresh tokens em texto puro em vez de hasheados, de modo que um vazamento do banco entrega sessões vivas.

## Recursos

- [RFC 6749: The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749) — a spec central.
- [RFC 7636: PKCE](https://datatracker.ietf.org/doc/html/rfc7636) — protegendo clientes públicos.
- [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/rfc9700) — orientação moderna de endurecimento.
- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519) — estrutura do token e claims.
- [OWASP: JWT for Java Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html) — erros comuns de validação.
