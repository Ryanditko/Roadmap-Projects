# Framework de Verificação de Qualidade de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um framework reutilizável que executa verificações de qualidade declarativas contra um dataset e reporta onde ele falha. Em vez de espalhar `assert` improvisados por todos os pipelines, você define regras uma vez — "esta coluna nunca é nula", "os totais dos pedidos são não-negativos", "customer_id é único" — e deixa o framework avaliá-las, pontuar o dataset e decidir se avisa ou interrompe o pipeline. É assim que as equipes param de enviar dados quebrados adiante. O desafio de design é tornar as regras expressivas o suficiente para serem úteis, baratas o suficiente para rodar em tabelas grandes e estruturadas o suficiente para que uma falha diga exatamente quais linhas e qual regra deram errado.

## Pré-requisitos

- Conforto para consultar ou varrer um dataset tabular (SQL, uma API de DataFrame ou arquivos)
- Entendimento das dimensões comuns de qualidade de dados: completude, validade, unicidade, consistência
- Familiaridade com a ideia de uma regra declarativa versus código imperativo
- Qualquer linguagem e um dataset com imperfeições realistas para testar

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Expressar regras de qualidade de forma declarativa e avaliá-las uniformemente
- Cobrir as dimensões centrais de qualidade — completude, validade, unicidade, consistência, frescor
- Produzir um relatório estruturado que nomeia a regra, as linhas afetadas e a contagem de falhas
- Calcular uma pontuação de qualidade do dataset e usá-la como portão de um pipeline
- Separar falhas bloqueantes de avisos para que dados ruins nem sempre parem a linha

## Requisitos Funcionais

1. O framework deve aceitar um conjunto de regras vinculadas a um dataset e colunas, definidas como dados/config, não lógica hardcoded.
2. Deve suportar verificações de não-nulo, unicidade, faixa/domínio, regex/formato e consistência entre colunas.
3. Cada verificação deve reportar aprovado/reprovado, o número de linhas violadoras e uma amostra dos valores ofensores.
4. Deve calcular uma pontuação geral de qualidade (ex.: taxa de aprovação ponderada) para o dataset.
5. As regras devem carregar uma severidade para que o framework possa avisar versus bloquear o pipeline.
6. Uma verificação referencial deve confirmar que chaves em um dataset existem em outro.
7. Os resultados devem ser persistidos por execução para que a qualidade possa ser acompanhada ao longo do tempo.

## Marcos Sugeridos

1. **Marco 1 — Motor de regras:** Defina um formato de regra e avalie verificações de coluna única, produzindo resultados de aprovado/reprovado.
2. **Marco 2 — Cobertura e relatórios:** Adicione verificações entre colunas, referenciais e estatísticas com um relatório estruturado e uma pontuação.
3. **Marco 3 — Portão e histórico:** Adicione severidades que barram o pipeline e persista resultados para acompanhar tendências de qualidade.

## Esboço de Dados e Interface

```text
rule
  id          string
  dataset     string
  column(s)   list
  type        enum(not_null | unique | range | regex | consistency | referential)
  params      map     (ex.: { min: 0 } ou { pattern: "..." })
  severity    enum(warn | block)

check_result
  rule_id, passed(bool), rows_total, rows_failed, sample_values[], run_id, ran_at

report
  dataset, score(0..100), results[], blocking_failures(int)
  -> se blocking_failures > 0: falhar o pipeline

exemplo de regra: { column: "email", type: regex, params: { pattern: forma name@example.com } }
```

## Desafios Extras

- Adicione verificações estatísticas: detecte uma média de coluna atípica ou um desvio de distribuição versus uma baseline.
- Sugira regras automaticamente perfilando uma amostra limpa (inferir tipos, faixas e taxas de nulos).
- Emita uma tendência de qualidade por execução e alerte quando a pontuação degradar entre execuções.
- Suporte colocar linhas com falha em quarentena numa tabela lateral em vez de bloquear toda a carga.

## Definição de Pronto

- [ ] As regras são definidas como configuração e rodam sem tocar no código do framework.
- [ ] Uma verificação reprovada reporta a regra, a contagem e uma amostra dos valores ruins reais.
- [ ] A pontuação de qualidade reflete o resultado ponderado de todas as verificações.
- [ ] Uma falha de severidade `block` para o pipeline; uma falha `warn` o deixa prosseguir.
- [ ] Uma verificação referencial captura corretamente uma chave estrangeira órfã.

## Armadilhas Comuns

- Escrever verificações como código pontual, de modo que nada seja reutilizável entre datasets — mantenha as regras declarativas.
- Reportar apenas aprovado/reprovado sem amostra, forçando quem faz a triagem a reconsultar a tabela inteira.
- Rodar linha a linha no código da aplicação quando uma consulta baseada em conjuntos seria muito mais barata em escala.
- Tratar toda falha como bloqueante, de modo que um problema cosmético interrompe uma carga crítica.
- Verificar unicidade ou nulos mas nunca validar regras de negócio entre colunas, onde os bugs reais se escondem.

## Recursos

- [Great Expectations: Conceitos centrais](https://docs.greatexpectations.io/docs/core/introduction/) — um framework declarativo maduro de qualidade de dados para estudar.
- [dbt: Testes](https://docs.getdbt.com/docs/build/data-tests) — como testes são declarados e barrados num fluxo de warehouse.
- [Wikipedia: Data quality](https://en.wikipedia.org/wiki/Data_quality) — as dimensões padrão de qualidade.
- [Artigo do Amazon Deequ](https://www.vldb.org/pvldb/vol11/p1781-schelter.pdf) — automatizando a verificação de qualidade de dados em larga escala.
