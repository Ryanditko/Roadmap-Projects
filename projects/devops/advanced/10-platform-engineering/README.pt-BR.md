# Kit de Engenharia de Plataforma

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa uma plataforma interna de desenvolvimento (IDP) que permite a um engenheiro de produto ir de "preciso de um novo serviço" a um deploy rodando, observável e conforme às políticas, sem abrir um ticket nem aprender toda a profundidade da sua infraestrutura. Você vai projetar templates de caminho dourado (golden path), uma interface de autoatendimento (portal, CLI ou baseada em Git) e a automação por trás dela que provisiona infraestrutura, conecta observabilidade e secrets e impõe políticas — tudo mantendo as proteções invisíveis até que alguém esbarre em uma. O verdadeiro desafio é pensamento de produto aplicado à infraestrutura: o nível certo de abstração, escotilhas de escape para os 10% de casos que o caminho dourado não cobre e adoção impulsionada por tornar a estrada pavimentada genuinamente mais rápida do que fazer à mão. Uma plataforma que ninguém usa é só mais infraestrutura para manter.

## Pré-requisitos

- Base sólida em Kubernetes, IaC e CI/CD (este projeto os compõe)
- Experiência implantando um serviço de ponta a ponta ao menos uma vez manualmente
- Familiaridade com templating (Helm/Kustomize) e conceitos de policy-as-code
- Entender o fluxo de trabalho do desenvolvedor que você pretende abstrair

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar templates de caminho dourado que codificam boas práticas por padrão
- Construir uma interface de autoatendimento que provisiona um serviço sem ops manual
- Impor policy-as-code para que a conformidade seja automática, não uma etapa de revisão
- Fornecer escotilhas de escape para casos que a estrada pavimentada não cobre
- Medir a adoção da plataforma e o tempo-até-o-primeiro-deploy do desenvolvedor

## Requisitos Funcionais

1. Um desenvolvedor deve conseguir criar um novo serviço a partir de um template via autoatendimento, sem ticket.
2. O provisionamento deve conectar observabilidade, secrets e CI/CD automaticamente.
3. Policy-as-code deve impor padrões (nomenclatura, limites de recursos, segurança) na criação e no deploy.
4. O caminho dourado deve ter uma escotilha de escape documentada para necessidades fora do padrão.
5. A plataforma deve expor o status dos serviços de um desenvolvedor (deploy, saúde) em um só lugar.
6. Integrar um novo serviço deve ser mensuravelmente mais rápido do que a linha de base manual.
7. A própria configuração da plataforma deve ser versionada e reproduzível.

## Marcos Sugeridos

1. **Marco 1 — Template de caminho dourado:** Defina um template que produz um serviço conforme e observável.
2. **Marco 2 — Interface de autoatendimento:** Permita a um desenvolvedor instanciar o template (portal, CLI ou PR no Git) sem ops.
3. **Marco 3 — Política e proteções:** Imponha policy-as-code e exponha violações com feedback claro.
4. **Marco 4 — Visibilidade e adoção:** Adicione um catálogo/visão de status de serviços e meça o tempo-até-o-primeiro-deploy vs. a linha de base.

## Esboço de Dados e Interface

```text
   desenvolvedor
      │  "novo serviço: checkout"  (portal / CLI / PR no Git)
      ▼
   ┌──────────────────┐   template de caminho dourado
   │ Interface de      │   (repo + CI + manifests + observabilidade conectada)
   │ autoatendimento   │
   └────────┬─────────┘
            │ orquestrar
            ▼
   ┌──────────────────┐    gate de policy-as-code (OPA/Kyverno)
   │ Orquestrador      │───▶ impor: nomenclatura, limites, segurança
   │ da plataforma     │    passa -> provisiona ; falha -> feedback claro
   └───┬────────┬──────┘
       ▼        ▼            ▼             ▼
   infra     observab.      secrets     pipeline CI/CD
   (IaC)     conectada      injetados   criado
       └────────┴────────────┴─────────────┘
                     ▼
              ┌───────────────┐
              │ Catálogo de    │  status: deploys, saúde, dono
              │ serviços       │
              └───────────────┘

Escotilha de escape: o caminho dourado cobre ~90%; documente como estender/optar por sair no resto.

Alvos não-funcionais:
  tempo-até-primeiro-deploy   minutos, não dias (medir vs. linha de base manual)
  conformidade de política    100% imposta na criação, não revisão a posteriori
  adoção                      % de novos serviços usando a estrada pavimentada, rastreada
```

## Desafios Extras

- Adicione múltiplos caminhos dourados (serviço sem estado, cron job, pipeline de dados) com blocos de construção compartilhados.
- Integre a plataforma a um portal de desenvolvedor real (Backstage) e a um catálogo de serviços.
- Adicione scorecards de custo e segurança por serviço para que os donos vejam sua postura.
- Suporte ambientes de preview efêmeros criados por pull request via os mesmos templates.

## Definição de Pronto

- [ ] Um desenvolvedor cria um serviço conforme e observável via autoatendimento sem ticket.
- [ ] Observabilidade, secrets e CI/CD são conectados automaticamente.
- [ ] Policy-as-code impõe padrões na criação e no deploy com feedback claro.
- [ ] Existe uma escotilha de escape documentada para casos fora do padrão.
- [ ] O tempo-até-o-primeiro-deploy é medido e supera a linha de base manual; a adoção é rastreada.

## Armadilhas Comuns

- Abstrair demais, de modo que a plataforma não consegue expressar os 10% de casos reais, forçando ferramentas paralelas.
- Construir a plataforma sem pesquisa com usuários e depois se perguntar por que ninguém adota.
- Tornar a estrada pavimentada mais lenta ou mais confusa do que fazer à mão — a adoção morre.
- Política que bloqueia com erros crípticos, treinando os desenvolvedores a contornar a plataforma.
- Tratar a plataforma como um projeto único em vez de um produto com usuários e um roadmap.

## Recursos

- [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/) — o que é uma plataforma e como raciocinar sobre ela.
- [Documentação do Backstage](https://backstage.io/docs/overview/what-is-backstage/) — um framework aberto de portal de desenvolvedor.
- [team-topologies.com](https://teamtopologies.com/) — times de plataforma e carga cognitiva.
- [Google SRE Workbook: Engagement](https://sre.google/workbook/engagement-model/) — plataforma-como-produto e pensamento de adoção.
