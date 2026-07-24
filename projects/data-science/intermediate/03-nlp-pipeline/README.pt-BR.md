# Pipeline de NLP (Tokenização + Embeddings)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Texto bruto não é algo que um modelo consiga consumir — ele precisa virar números primeiro. Este projeto constrói o pipeline que faz essa conversão de ponta a ponta: limpar e tokenizar documentos, transformar tokens em vetores (de uma matriz esparsa TF-IDF até embeddings densos) e empacotar o resultado para que um classificador ou índice de busca posterior possa usá-lo. O objetivo não é treinar o modelo mais sofisticado, mas construir a camada de transformação reutilizável e bem testada que todo projeto de texto precisa, e provar que suas representações de fato capturam significado medindo-as numa pequena tarefa posterior.

## Pré-requisitos

- Conforto com manipulação de strings em Python e uma biblioteca de dataframe
- Entender o que é um vetor e a similaridade por cosseno
- Familiaridade com divisão treino/teste para avaliação
- Um conjunto de texto com rótulos (ex.: reviews de sentimento ou notícias com tópicos)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir um estágio de pré-processamento configurável (minúsculas, tokenização, stopwords, lematização)
- Produzir vetores de documento esparsos (TF-IDF) e densos (Word2Vec/GloVe ou transformer)
- Ajustar o vetorizador só no texto de treino e transformar validação/teste com ele
- Medir a qualidade do embedding numa tarefa de classificação posterior, não só no olho
- Tratar tokens fora do vocabulário (OOV) e documentar o vocabulário que você mantém

## Requisitos Funcionais

1. O pipeline deve aceitar documentos brutos e emitir uma matriz de features documentada e reutilizável.
2. Os passos de pré-processamento devem ser individualmente ligáveis/desligáveis e seu efeito observável.
3. O vetorizador deve ser ajustado só no split de treino e então aplicado aos dados retidos.
4. Deve oferecer pelo menos duas representações (TF-IDF e uma baseada em embedding) para comparação.
5. Deve avaliar cada representação no mesmo classificador posterior e reportar métricas.
6. Os casos de OOV e documento vazio devem ser tratados sem quebrar.
7. O tamanho e a cobertura do vocabulário devem ser reportados.

## Marcos Sugeridos

1. **Marco 1 — Pré-processar:** Tokenize, normalize e construa um vocabulário limpo.
2. **Marco 2 — Vetorizar:** Produza representações TF-IDF e por embedding.
3. **Marco 3 — Avaliar:** Treine um classificador simples em cada uma e compare as métricas.

## Esboço de Dados e Interface

```text
Registro de documento
  doc_id : string
  text   : string bruta
  label  : categoria   (para avaliação posterior)

Passos do pipeline
  1. dividir docs -> treino / valid / teste
  2. pré-processar: normalizar -> tokenizar -> stopwords -> lematizar
  3. ajustar vetorizador nos tokens de TREINO
       tfidf  -> matriz esparsa (n_docs x vocab)
       embed  -> vetores de palavra agregados -> denso (n_docs x dim)
  4. transformar valid/teste com o vetorizador ajustado
  5. avaliar: LogisticRegression em cada -> acurácia / macro-F1
  6. reportar tamanho do vocab, taxa de OOV, cobertura
```

## Desafios Extras

- Adicione tokenização por subpalavras (BPE/WordPiece) e meça seu efeito na taxa de OOV.
- Troque por embeddings contextuais de um transformer pré-treinado e compare custo vs ganho.
- Visualize embeddings em 2D (UMAP/t-SNE) coloridos por rótulo para inspecionar a separabilidade.
- Faça cache dos tokens pré-processados para que reexecutar a vetorização não re-tokenize.

## Definição de Pronto

- [ ] O pipeline transforma texto bruto em matriz de features em uma chamada documentada.
- [ ] O vetorizador é ajustado só nos dados de treino — sem vazar vocabulário do teste.
- [ ] Duas representações são comparadas na mesma tarefa de classificação retida.
- [ ] OOV e documentos vazios são tratados graciosamente com uma política definida.
- [ ] O tamanho do vocabulário e a taxa de OOV são reportados.

## Armadilhas Comuns

- Ajustar TF-IDF no corpus inteiro, vazando o vocabulário de teste e inflando os scores.
- Limpar texto demais (removendo negações, emojis) e destruir o sinal do qual o rótulo depende.
- Fazer média de vetores de palavra sem tratar documentos onde todo token é OOV.
- Comparar representações com classificadores diferentes, sem conseguir isolar a causa.

## Recursos

- [spaCy 101](https://spacy.io/usage/spacy-101) — tokenização, lematização e pipelines.
- [scikit-learn: extração de features de texto](https://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction) — TF-IDF em profundidade.
- [Jay Alammar: The Illustrated Word2Vec](https://jalammar.github.io/illustrated-word2vec/) — como embeddings de palavras funcionam.
- [Documentação do Sentence Transformers](https://www.sbert.net/) — embeddings densos modernos de documentos.
