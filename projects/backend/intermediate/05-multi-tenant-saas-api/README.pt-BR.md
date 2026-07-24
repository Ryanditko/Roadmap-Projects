# Multi-Tenant SaaS API

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

A maioria dos produtos SaaS atende muitos clientes — os tenants — a partir de uma única aplicação em execução, prometendo a cada um deles que seus dados são completamente invisíveis para os outros. Este projeto pede que você construa a maquinaria que torna essa promessa verdadeira. Uma requisição chega, você descobre a qual tenant ela pertence, anexa essa identidade a tudo que vem depois, e cada consulta que você roda é silenciosamente restringida para que um tenant nunca leia ou escreva os dados de outro. A parte difícil não é a lista de funcionalidades — é que um único filtro esquecido vira um vazamento de dados, então o isolamento precisa ser o padrão, não algo que cada handler precisa lembrar de fazer.

## Pré-requisitos

- Conforto para construir e proteger uma API REST com autenticação ([API de E-commerce com Autenticação JWT](../01-ecommerce-api-jwt/) é um bom precursor)
- Um banco de dados relacional e conhecimento prático de schemas, índices e cláusulas `WHERE`
- Entendimento de middleware / hooks do ciclo de vida da requisição no seu framework
- Familiaridade com como o contexto da requisição é propagado (thread-locals, contexto assíncrono ou passagem explícita)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Resolver um tenant a partir de um subdomínio, cabeçalho ou token e rejeitar requisições que não resolvem nenhum
- Comparar modelos de isolamento — schema compartilhado com coluna de tenant, schema-por-tenant, banco-por-tenant — e justificar uma escolha
- Impor o isolamento de dados de modo que esquecer um filtro falhe com segurança em vez de vazar
- Propagar o contexto do tenant de forma limpa desde a borda até a camada de dados
- Provisionar e desprovisionar tenants como um ciclo de vida de primeira classe
- Alternar funcionalidades e cotas por tenant sem fazer redeploy

## Requisitos Funcionais

1. O sistema deve resolver toda requisição autenticada para exatamente um tenant (via subdomínio, cabeçalho ou uma claim dentro do token) e retornar 400/401 quando não conseguir.
2. Um tenant nunca deve conseguir ler, atualizar ou excluir registros de outro tenant, nem mesmo adivinhando IDs.
3. O acesso a dados deve ser restringido centralmente para que um novo endpoint seja isolado por padrão, sem código por handler.
4. O sistema deve expor um endpoint de onboarding que provisiona um novo tenant e seu admin inicial.
5. O sistema deve suportar feature flags por tenant que mudam o comportamento sem um deploy.
6. O sistema deve registrar o uso por tenant (requisições, armazenamento ou uma métrica de domínio) para cobrança futura.
7. A exclusão de um tenant deve remover ou anonimizar irreversivelmente os dados daquele tenant.

## Marcos Sugeridos

1. **Marco 1 — Resolução de tenant:** Extraia o tenant da requisição, carregue-o e rejeite tenants desconhecidos ou inativos no middleware.
2. **Marco 2 — Isolamento imposto:** Escolha um modelo de isolamento e roteie todas as leituras/escritas por uma camada restringida; prove que uma leitura entre tenants é impossível.
3. **Marco 3 — Ciclo de vida e flags:** Adicione onboarding de tenant, feature flags por tenant, medição de uso e exclusão.

## Esboço de Dados e Interface

```text
Tenant
  id:        uuid
  slug:      string   (subdomínio, ex.: "acme")
  status:    enum(active, suspended, deleting)
  plan:      string
  features:  map<string, bool>
  createdAt: string ISO-8601

RegistroComEscopoDeTenant (toda tabela de negócio)
  id:        uuid
  tenantId:  uuid   (indexado, nunca fornecido pelo cliente)
  ...campos de domínio

Resolução: Host "acme.api.example.com" | cabeçalho X-Tenant-Id | claim JWT "tid"
           -> carrega Tenant -> anexa ao contexto da requisição

POST /tenants               -> 201 { id, slug }           (provisiona)
GET  /projects              -> 200 [ ... ]                 (escopo automático ao tenant)
GET  /admin/usage           -> 200 { requests, storageMb } (por tenant)
DELETE /tenants/{id}        -> 202  (purga assíncrona)

Opções de isolamento: schema compartilhado + filtro tenantId (+ RLS do Postgres),
                      schema-por-tenant, banco-por-tenant
```

## Desafios Extras

- Imponha o isolamento no próprio banco de dados com Row-Level Security do PostgreSQL, para que um filtro faltando não vaze.
- Adicione rate limiting e imposição de cotas por tenant atrelados ao plano.
- Ofereça uma exportação de dados self-service ("baixar todos os meus dados") por tenant.
- Suporte uma proteção contra "vizinho barulhento": limite os recursos que qualquer tenant pode consumir.

## Definição de Pronto

- [ ] Toda requisição sem um tenant resolvível e ativo é rejeitada antes de chegar à lógica de negócio.
- [ ] Um teste prova que o tenant A não consegue buscar um registro do tenant B mesmo com um ID válido.
- [ ] Adicionar um novo endpoint não exige código extra para ser isolado por tenant.
- [ ] Um tenant pode ser provisionado e excluído de ponta a ponta, com a exclusão removendo seus dados.
- [ ] Ao menos uma feature flag muda o comportamento para um tenant e não para os outros.

## Armadilhas Comuns

- Depender de cada handler lembrar do filtro `tenantId` — um único `WHERE` esquecido é uma brecha. Restrinja centralmente.
- Confiar em um tenant ID fornecido pelo cliente no corpo enquanto o token diz outra coisa; sempre derive-o no servidor.
- Vazar a existência de um tenant por mensagens de erro ou enumeração de IDs (diferenças entre 404 e 403).
- Fazer cache sem namespacing das chaves por tenant, servindo a resposta cacheada de um tenant para outro.
- Rodar migrações em tabelas compartilhadas sem considerar tenants em pleno provisionamento.

## Recursos

- [Microsoft: Padrões de tenancy de banco de dados SaaS multi-tenant](https://learn.microsoft.com/pt-br/azure/azure-sql/database/saas-tenancy-app-design-patterns) — a comparação canônica dos modelos de isolamento.
- [PostgreSQL: Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html) — imponha o isolamento de tenant no próprio banco.
- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) — prevenindo a classe de vazamento por broken-object-level-authorization.
- [Stripe: Projetando APIs robustas e previsíveis com idempotência](https://stripe.com/blog/idempotency) — contexto útil para fluxos de provisionamento de tenant.
