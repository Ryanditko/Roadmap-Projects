# Sistema de Ensemble Multi-Modelo

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um sistema que combina vários modelos em um ensemble que supera qualquer um deles sozinho — e, tão importante quanto, sabe quando não supera. Fazer ensemble não é "tirar a média de alguns modelos"; seu poder vem da diversidade, e combinar modelos correlacionados não compra nada enquanto multiplica o custo de inferência. Neste projeto você monta um pool diverso de modelos, implementa estratégias reais de combinação (stacking com meta-learner, blending ponderado, votação), mede a diversidade honestamente e pesa o ganho de acurácia contra a computação e a latência que você paga por ele. A tensão interessante é que o melhor ensemble raramente é "todos eles".

## Pré-requisitos

- Domínio sólido de viés–variância e por que erros diversos se cancelam
- Experiência treinando múltiplas famílias de modelos e fazendo validação cruzada adequada
- Entendimento de stacking, blending e o risco de vazamento no meta-learner
- Conforto para medir custo de inferência, não apenas acurácia

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir um pool deliberadamente diverso de modelos base e medir sua diversidade
- Implementar stacking com predições out-of-fold para evitar vazamento no meta-learner
- Comparar stacking, blending ponderado e votação na mesma tarefa
- Pesar os ganhos de acurácia do ensemble contra a latência e computação adicionadas
- Podar o ensemble para o subconjunto que carrega a maior parte do benefício

## Requisitos Funcionais

1. O sistema deve treinar e gerenciar um pool de ao menos três modelos base diversos.
2. Deve calcular predições out-of-fold para que o meta-learner nunca veja vazamento in-fold.
3. Deve implementar ao menos duas estratégias de combinação (ex.: stacking e blending ponderado).
4. Deve medir a diversidade do ensemble (ex.: discordância par a par ou correlação de erros).
5. Deve reportar o desempenho do ensemble contra o melhor modelo único e uma média ingênua.
6. Deve expor o ensemble atrás de uma interface de predição com a contribuição por modelo visível.
7. Deve suportar a poda do pool para um subconjunto menor com um tradeoff declarado de acurácia/custo.

## Requisitos Não Funcionais

- **Acurácia vs custo:** o ganho de acurácia do ensemble deve ser reportado junto de sua latência/computação adicionada.
- **Latência:** a inferência total do ensemble deve permanecer dentro de um orçamento declarado (modelos base podem rodar em paralelo).
- **Reprodutibilidade:** atribuições de fold, sementes e pesos devem reproduzir os resultados reportados.
- **Robustez:** a falha de um modelo base deve degradar o ensemble graciosamente, não quebrá-lo.

## Marcos Sugeridos

1. **Marco 1 — Pool diverso:** Treine várias famílias de modelos distintas e meça sua diversidade par a par.
2. **Marco 2 — Stacking:** Gere predições out-of-fold e treine um meta-learner sem vazamento.
3. **Marco 3 — Comparar estratégias:** Adicione blending ponderado e votação; compare contra o melhor-único e a média-ingênua.
4. **Marco 4 — Podar e servir:** Pode para um subconjunto custo-efetivo e sirva com contribuições por modelo.

## Esboço de Dados e Interface

```text
 dados de treino
      |
      +--> Modelo A (árvores)    \
      +--> Modelo B (boosting)    >  pool base (diverso!)
      +--> Modelo C (rede neural)/
      +--> Modelo D (linear)    /
             |
             v   predições out-of-fold (sem vazamento)
   +-----------------------------+
   | Matriz de predições OOF      |   linhas=amostras, cols=modelos base
   +--------------+--------------+
                  v
   +------ combinar (escolha) ----+
   | stacking:  meta-learner(OOF) |
   | blending:  média ponderada   |
   | votação:   maioria/soft      |
   +--------------+--------------+
                  v
   diversidade: discordância par a par, matriz de correlação de erros
   relatório: ensemble vs melhor-único vs média-ingênua  (acc E latência)

 GET /predict -> { prediction, per_model: {A:.., B:..}, weights }
```

## Desafios Extras

- Adicione ponderação dinâmica por instância (escolha especialistas com base na região da entrada).
- Adicione atualização online para que os pesos se adaptem conforme chegam novos dados rotulados.
- Adicione explicabilidade que atribui a predição final de volta aos modelos contribuintes.
- Busque o subconjunto ótimo com um método guloso ou evolutivo de seleção de ensemble.

## Definição de Pronto

- [ ] O pool base tem diversidade medida, não apenas múltiplas cópias do mesmo modelo.
- [ ] O stacking usa predições out-of-fold sem vazamento no meta-learner.
- [ ] Ao menos duas estratégias de combinação são comparadas contra as baselines de melhor-único e média-ingênua.
- [ ] Os ganhos de acurácia são reportados junto do custo de latência/computação adicionado.
- [ ] Um subconjunto podado é oferecido com um tradeoff explícito de acurácia-vs-custo.

## Armadilhas Comuns

- Construir um ensemble "diverso" de modelos altamente correlacionados que adiciona custo mas nenhuma acurácia.
- Treinar o meta-learner em predições in-fold e vazar, inflando os ganhos aparentes.
- Reportar apenas o ganho de acurácia e esconder que a inferência agora custa 5x mais.
- Assumir que mais modelos é sempre melhor em vez de podar para o subconjunto efetivo.

## Recursos

- [scikit-learn: métodos de Ensemble](https://scikit-learn.org/stable/modules/ensemble.html) — APIs e teoria de stacking, votação e boosting.
- [Stacked Generalization (Wolpert, 1992)](https://www.sciencedirect.com/science/article/abs/pii/S0893608005800231) — o paper original de stacking.
- [Ensemble Selection from Libraries of Models (Caruana et al., 2004)](https://dl.acm.org/doi/10.1145/1015330.1015432) — poda gulosa de ensemble.
- [Popular Ensemble Methods: An Empirical Study (Opitz & Maclin, 1999)](https://www.jair.org/index.php/jair/article/view/10239) — por que a diversidade impulsiona os ganhos de ensemble.
