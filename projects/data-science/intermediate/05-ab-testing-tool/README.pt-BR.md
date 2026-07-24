# Ferramenta de Análise de Testes A/B

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Alguém rodou um experimento: a versão A tinha um botão, a versão B tinha outro, e agora há uma planilha de conversões. B realmente venceu, ou a diferença é ruído? Este projeto constrói a ferramenta que responde a essa pergunta com rigor estatístico em vez de achismo. Você vai calcular o tamanho de amostra que um teste *deveria* ter tido antes de começar, rodar o teste de significância certo para o tipo de métrica, reportar um intervalo de confiança e um tamanho de efeito em vez de um p-valor solto, e se proteger das formas clássicas de um experimento mentir — espiar cedo, testar muitas métricas e confundir "não significativo" com "sem efeito".

## Pré-requisitos

- Entender médias, proporções, variância e a distribuição normal
- Familiaridade com a ideia de teste de hipóteses (nula vs alternativa)
- Conforto com uma biblioteca de estatística (SciPy, statsmodels) e uma ferramenta de dataframe
- Dados de experimento com um rótulo de grupo e um resultado por unidade

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Calcular o tamanho de amostra necessário a partir de taxa base, efeito mínimo detectável, poder e alfa
- Escolher o teste correto para a métrica (z-test de duas proporções, t-test de Welch, qui-quadrado)
- Reportar um intervalo de confiança e tamanho de efeito, não só um p-valor
- Aplicar uma correção de comparações múltiplas quando várias métricas são avaliadas
- Explicar por que espiar resultados cedo infla a taxa de falsos positivos

## Requisitos Funcionais

1. A ferramenta deve calcular o tamanho de amostra necessário dados base, MDE, poder e nível de significância.
2. Deve selecionar e rodar o teste apropriado conforme a métrica seja uma taxa ou uma média.
3. Deve produzir uma estimativa pontual, intervalo de confiança, tamanho de efeito e p-valor juntos.
4. Deve aplicar uma correção (Bonferroni ou Benjamini-Hochberg) quando mais de uma métrica é testada.
5. Deve sinalizar quando a amostra observada está abaixo do tamanho necessário e avisar sobre baixo poder.
6. Deve incluir pelo menos uma checagem de métrica de guardrail junto à métrica principal.
7. Deve produzir um veredito em linguagem simples ("ganho significativo de X% [IC]" ou "inconclusivo").

## Marcos Sugeridos

1. **Marco 1 — Poder e dimensionamento:** Implemente os cálculos de tamanho de amostra e poder.
2. **Marco 2 — Teste:** Rode o teste de significância correto com IC e tamanho de efeito.
3. **Marco 3 — Rigor:** Adicione correção de comparações múltiplas e checagens de guardrail/espiar.

## Esboço de Dados e Interface

```text
Registro do experimento (um por unidade)
  unit_id : string
  group   : "control" | "variant"
  metric  : float | 0/1 (conversão)

Saída da análise
  n_por_grupo, taxas/médias observadas
  teste_usado   : "z 2-prop" | "welch t" | "chi2"
  estimativa    : diferença de taxas/médias
  ic_95         : [baixo, alto]
  tamanho_efeito: h de Cohen / d
  p_valor, p_corrigido
  veredito      : "significativo" | "inconclusivo"
  aviso_poder   : bool

Passos
  1. n_necessario = f(base, mde, poder=0.8, alfa=0.05)
  2. escolher teste pelo tipo de métrica
  3. calcular estimativa, IC, tamanho de efeito, p
  4. corrigir p entre as métricas
  5. renderizar veredito + aviso de poder
```

## Desafios Extras

- Adicione uma análise sequencial/Bayesiana para que olhares antecipados sejam válidos por design.
- Rode um teste A/A em dados reais para confirmar que a taxa de falso positivo bate com o alfa.
- Suporte métricas de razão (ex.: receita por usuário) com o método delta para a variância.
- Adicione um modo de simulação que gera dados com efeito conhecido para validar a ferramenta.

## Definição de Pronto

- [ ] O tamanho de amostra é calculado a partir de entradas declaradas e mostrado antes de qualquer conclusão.
- [ ] O teste escolhido combina com o tipo de métrica e sua suposição é verificada.
- [ ] Todo resultado carrega um intervalo de confiança e tamanho de efeito, não só um p-valor.
- [ ] Múltiplas métricas disparam uma correção e os p-valores corrigidos são reportados.
- [ ] O veredito é declarado em linguagem simples, com a ressalva quando o poder é baixo.

## Armadilhas Comuns

- Reportar um p-valor sem tamanho de efeito, fazendo uma diferença trivial parecer importante.
- Testar dez métricas a alfa 0.05 e comemorar a única que "venceu" por acaso.
- Tratar p > 0.05 como prova de ausência de diferença em vez de evidência insuficiente.
- Usar um t-test numa métrica de conversão binária onde cabe um teste de proporção.

## Recursos

- [Kohavi et al.: Trustworthy Online Controlled Experiments](https://experimentguide.com/) — a referência definitiva para praticantes.
- [statsmodels: poder e tamanho de amostra](https://www.statsmodels.org/stable/stats.html#power-and-sample-size-calculations) — referência de implementação.
- [Wikipedia: problema de comparações múltiplas](https://en.wikipedia.org/wiki/Multiple_comparisons_problem) — por que correções importam.
- [Evan Miller: How Not to Run an A/B Test](https://www.evanmiller.org/how-not-to-run-an-ab-test.html) — o problema de espiar explicado.
