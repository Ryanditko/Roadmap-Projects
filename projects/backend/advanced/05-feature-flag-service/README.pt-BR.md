# Serviço de Feature Flags

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa o serviço que alimenta rollouts controlados e testes A/B em empresas como a LaunchDarkly ou nas plataformas internas da Netflix e do Facebook. Um serviço de feature flags permite que times liguem ou desliguem funcionalidades — e as direcionem a usuários específicos — sem fazer deploy de código novo. A parte interessante não é o interruptor liga/desliga; é o motor de avaliação que precisa resolver uma flag para um dado usuário em bem menos de um milissegundo, a partir de milhares de serviços, de forma consistente, mesmo quando a rede até o seu control plane está fora. Você projetará um modelo de regras de segmentação, um caminho de entrega de baixa latência e os trade-offs entre atualidade em tempo real e disponibilidade que tornam um sistema de flags confiável o bastante para proteger um fluxo de pagamento.

## Pré-requisitos

- Design sólido de APIs REST e experiência modelando domínios não triviais
- Familiaridade com estratégias de cache e os trade-offs de invalidação de cache
- Entendimento de percentis (latência p99) e por que a latência de cauda importa
- Estatística básica para testes A/B (tamanho de amostra, significância) é útil
- Conforto para raciocinar sobre consistência vs. disponibilidade sob partição

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar flags, variações e regras de segmentação como dados avaliáveis
- Construir um motor de avaliação determinístico que resolve regras em uma ordem fixa
- Projetar um caminho de entrega (SDK + cache/stream) que sobrevive a quedas do control plane
- Implementar rollouts percentuais com bucketing de usuário estável e "grudento" (sticky)
- Raciocinar sobre consistência de avaliação entre muitos serviços e instâncias
- Emitir eventos de exposição que alimentam a análise de testes A/B sem desacelerar a avaliação

## Requisitos Funcionais

1. O sistema deve permitir que um operador defina uma flag com um valor padrão e uma ou mais variações (booleana ou multivariada).
2. O sistema deve avaliar uma flag contra um contexto de usuário (atributos como id, plano, país) e retornar exatamente uma variação de forma determinística.
3. Regras de segmentação devem ser avaliadas em uma ordem definida e documentada; a primeira regra que casar vence, senão o padrão se aplica.
4. Rollouts percentuais devem ser sticky: o mesmo usuário deve sempre cair no mesmo bucket, a menos que a regra mude.
5. O caminho de avaliação deve servir leituras com p99 bem abaixo de 5 ms e deve continuar servindo o último ruleset conhecido se o control plane estiver inacessível (disponibilidade sobre atualidade).
6. Mudanças de flag devem se propagar aos avaliadores dentro de uma janela de defasagem limitada e documentada (ex.: transmitida em segundos, ou por polling).
7. Toda mudança de flag deve ser registrada em uma trilha de auditoria imutável (quem, o quê, quando).
8. O sistema deve expor um kill switch que força uma flag ao seu padrão instantaneamente em todos os consumidores.
9. A avaliação deve emitir eventos de exposição (usuário, flag, variação) para analytics sem bloquear a resposta.

## Marcos Sugeridos

1. **Marco 1 — Modelo de flag e CRUD:** Defina flags com variações e padrões; construa a API de gerenciamento e o log de auditoria.
2. **Marco 2 — Motor de avaliação:** Implemente regras de segmentação ordenadas e resolução determinística de variação única contra um contexto de usuário.
3. **Marco 3 — Rollouts sticky:** Adicione rollouts percentuais usando um hash de `flagKey + userId` para bucketing estável.
4. **Marco 4 — Entrega e resiliência:** Adicione um cache/stream no cliente para que avaliadores sirvam localmente e sobrevivam à indisponibilidade do control plane.
5. **Marco 5 — Exposição e experimentos:** Emita eventos de exposição de forma assíncrona e exponha métricas A/B básicas.

## Esboço de Dados e Interface

```text
Flag
  key:         string
  type:        booleana | multivariada
  variations:  [ v0, v1, ... ]
  default:     variationIndex
  rules:       [ { conditions:[...], variation | rollout } ]  (ordenadas)
  version:     inteiro

Fluxo de avaliação
  SDK do Consumidor --(poll/stream do ruleset)--> API do Control Plane
       |  (avaliação local, em memória, sem rede no caminho quente)
       v
  evaluate(flagKey, userContext) -> { variation, ruleMatched, reason }
       |
       +--async--> Barramento de Eventos de Exposição --> Store de Analytics

Rollout sticky:  bucket = hash(flagKey + ":" + userId) % 100
                 no rollout se bucket < rolloutPercentage

POST /flags                 -> 201 flag
PATCH /flags/{key}          -> 200 (nova versão, entrada de auditoria)
POST /evaluate              body {flagKey, userContext} -> 200 {variation, reason}
POST /flags/{key}/kill      -> 200 (força o padrão em todo lugar)
```

## Desafios Extras

- Adicione flags pré-requisito (a flag B só avalia se a flag A estiver ligada).
- Suporte segmentos de usuário como fragmentos de regra reutilizáveis e nomeados.
- Adicione rollouts agendados que aumentam um percentual ao longo do tempo automaticamente.
- Calcule significância A/B e métricas de guardrail a partir dos eventos de exposição.
- Ofereça um modo de entrega em streaming (SSE) e compare sua defasagem vs. polling.

## Definição de Pronto

- [ ] O mesmo contexto de usuário sempre resolve para a mesma variação para um ruleset fixo.
- [ ] A ordem das regras é determinística e documentada; o padrão é retornado quando nada casa.
- [ ] Rollouts percentuais são sticky entre reinícios e instâncias.
- [ ] Avaliadores servem o último ruleset conhecido quando o control plane está fora.
- [ ] Toda mudança é capturada na trilha de auditoria e o kill switch força os padrões instantaneamente.
- [ ] Eventos de exposição são emitidos sem adicionar latência à avaliação.

## Armadilhas Comuns

- Avaliar flags pela rede a cada requisição — o caminho quente deve ser local e em memória.
- Fazer bucketing sobre um número aleatório em vez de um hash estável, fazendo usuários oscilarem entre variações.
- Falhar fechado (dando erro) quando o control plane está fora, em vez de falhar para o último valor conhecido.
- Vazar a avaliação para o analytics de forma síncrona, acoplando a latência de resposta ao seu pipeline de eventos.
- Mutar flags no lugar sem versão ou auditoria, tornando um rollout ruim impossível de explicar.
- Ignorar a cardinalidade dos atributos de usuário na segmentação, deixando as regras explodirem em custo.

## Recursos

- [Martin Fowler: Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) — a taxonomia canônica dos tipos de flag.
- [Especificação OpenFeature](https://openfeature.dev/specification/) — um padrão neutro de fornecedor para avaliação de flags e SDKs.
- [Blog da LaunchDarkly](https://launchdarkly.com/blog/) — artigos práticos de engenharia sobre entrega e consistência.
- [Consistent hashing (Wikipedia)](https://en.wikipedia.org/wiki/Consistent_hashing) — base para bucketing estável e bem distribuído.
- [Trustworthy Online Controlled Experiments (Kohavi et al.)](https://experimentguide.com/) — a referência sobre rodar testes A/B corretamente.
