# Análise de Sentimento em Texto

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Análise de sentimento transforma texto livre — uma avaliação, um tuíte, um ticket de suporte — em um rótulo como positivo ou negativo. É um primeiro passo suave no Processamento de Linguagem Natural porque o pipeline é concreto: limpe o texto, transforme palavras em números, treine um classificador e veja onde ele erra. Neste projeto você constrói isso de ponta a ponta sobre um conjunto rotulado de avaliações, e dedica tempo de verdade aos casos de erro, porque as lições interessantes de PLN vivem nas classificações erradas — negação, sarcasmo e gírias de domínio que um modelo bag-of-words simplesmente não enxerga.

## Pré-requisitos

- Python básico e pandas
- scikit-learn instalado
- Entendimento do que um classificador faz (mapeia features a um rótulo)
- Um conjunto de texto rotulado — as [avaliações de filmes do IMDb](https://ai.stanford.edu/~amaas/data/sentiment/) ou o [dataset Sentiment Labelled Sentences da UCI](https://archive.ics.uci.edu/dataset/331/sentiment+labelled+sentences) são boas escolhas

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Pré-processar texto bruto (minúsculas, tokenização, tratamento de stopwords)
- Converter texto em features numéricas com uma representação bag-of-words ou TF-IDF
- Treinar e avaliar um classificador de texto (Naive Bayes ou Regressão Logística)
- Inspecionar classificações erradas para entender as limitações do modelo
- Explicar por que um modelo linear bag-of-words tropeça em negação e sarcasmo

## Requisitos Funcionais

1. O fluxo deve carregar um conjunto de texto rotulado e relatar o balanceamento de classes.
2. Deve aplicar um pipeline de pré-processamento documentado ao texto bruto.
3. Deve vetorizar o texto em features numéricas (bag-of-words ou TF-IDF).
4. Deve treinar um classificador e avaliá-lo em um conjunto de teste separado.
5. Deve relatar acurácia, precisão, recall e F1, além de uma matriz de confusão.
6. Deve trazer à tona e discutir ao menos três exemplos classificados errado.
7. Deve prever o sentimento de uma nova frase escrita à mão.

## Marcos Sugeridos

1. **Marco 1 — Pré-processar e vetorizar:** Limpe o texto e transforme-o em uma matriz de features TF-IDF.
2. **Marco 2 — Treinar e avaliar:** Ajuste um classificador, relate métricas e construa a matriz de confusão.
3. **Marco 3 — Análise de erros:** Examine classificações erradas, identifique padrões e teste com suas próprias frases.

## Esboço de Dados e Interface

```text
Pipeline do modelo (texto -> rótulo)
  texto bruto     "This movie was NOT good at all."
    -> preprocess  minúsculas, remove pontuação, tokeniza, (opcional) remove stopwords
    -> vetoriza    TF-IDF -> vetor esparso [n_features]
    -> classifica  -> rótulo em {positive, negative}  (+ probabilidade)

Formato dos dados
  entrada: { text: string, label: "positive" | "negative" }
  vetor:   cada token -> uma coluna ponderada; documento -> uma linha de pesos

Tabela de análise de erros
  texto                       | real      | previsto  | causa provável
  "not good at all"           | negative  | positive  | negação perdida no bag-of-words
  "yeah, great, another bug"  | negative  | positive  | sarcasmo
```

## Desafios Extras

- Adicione bigramas para que "not good" vire uma única feature e compare as métricas.
- Compare TF-IDF com contagens simples de palavras no mesmo classificador.
- Adicione um limiar de probabilidade para marcar previsões de baixa confiança como "incerto".
- Experimente um modelo de sentimento pré-treinado (ex.: um pipeline do Hugging Face) e compare com o seu.

## Definição de Pronto

- [ ] Os passos de pré-processamento estão documentados e aplicados de forma consistente ao treino e ao teste.
- [ ] O texto é vetorizado e um classificador é treinado apenas na divisão de treino.
- [ ] Acurácia, precisão, recall, F1 e uma matriz de confusão são todos relatados.
- [ ] Ao menos três classificações erradas são mostradas com uma explicação plausível.
- [ ] O modelo classifica uma nova frase escrita à mão de ponta a ponta.

## Armadilhas Comuns

- Ajustar o vetorizador no conjunto completo, vazando vocabulário de teste para o treino.
- Remover stopwords cegamente — "not" e "no" carregam o sentimento que você quer.
- Relatar só acurácia em um conjunto desbalanceado que um chute constante venceria.
- Esperar que um modelo bag-of-words capture sarcasmo ou ordem de palavras que ele fundamentalmente ignora.

## Recursos

- [scikit-learn: Working with text data](https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html) — o tutorial canônico de classificação de texto.
- [scikit-learn: TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html) — transformando texto em features.
- [NLTK Book, Chapter 6: Text Classification](https://www.nltk.org/book/ch06.html) — fundamentos de pré-processamento e classificação.
- [Hugging Face: Sentiment analysis pipeline](https://huggingface.co/docs/transformers/main/en/quicktour) — um baseline pré-treinado para o desafio extra.
