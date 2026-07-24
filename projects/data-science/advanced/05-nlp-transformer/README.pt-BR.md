# Sistema de NLP com Transformer

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um serviço de NLP de nível de produção em torno de um transformer pré-treinado, com fine-tuning para uma tarefa concreta (classificação, extração ou sumarização) e servido sob um orçamento real de latência e memória. A engenharia interessante vive nas bordas do modelo: tokenização e truncamento eficientes para entradas longas, fine-tuning eficiente em parâmetros para treinar em hardware modesto, inferência quantizada para caber na memória e avaliação honesta que inclui um olhar sobre viés e modos de falha. Você vai fazer fine-tuning, servir, medir e interrogar o modelo — não apenas chamar uma API.

## Pré-requisitos

- Entendimento da arquitetura transformer (atenção, tokenização, embeddings)
- Experiência com a biblioteca `transformers` da Hugging Face
- Familiaridade com um dataset de texto rotulado para a tarefa escolhida
- Conforto com restrições de memória de GPU e treino em precisão mista

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Fazer fine-tuning de um transformer pré-treinado para uma tarefa específica, incluindo abordagens completa e eficiente em parâmetros (LoRA)
- Lidar com tokenização, truncamento e entradas longas corretamente
- Otimizar a inferência via quantização, batching e configurações de decodificação apropriadas
- Servir o modelo atrás de uma API dentro de um orçamento de latência e memória
- Avaliar a qualidade da tarefa junto com viés, calibração e modos de falha

## Requisitos Funcionais

1. O sistema deve fazer fine-tuning de um transformer pré-treinado em um dataset rotulado para uma tarefa.
2. A tokenização deve lidar com truncamento/padding e entradas que excedem a janela de contexto do modelo.
3. O fine-tuning deve suportar um método eficiente em parâmetros (ex.: LoRA) como alternativa ao fine-tuning completo.
4. A inferência deve ser quantizada e em batch, com parâmetros de decodificação configuráveis quando relevante.
5. O modelo deve ser servido por uma API com schema de requisição/resposta documentado.
6. A avaliação deve reportar métricas da tarefa mais ao menos uma sonda de viés/justiça.
7. A qualidade do modelo otimizado (quantizado) deve ser comparada com a versão em precisão total.

## Requisitos Não Funcionais

- **Latência:** p95 do serving dentro de um orçamento declarado no tamanho de batch alvo.
- **Memória:** o modelo servido deve caber dentro de um teto de memória GPU/CPU declarado.
- **Reprodutibilidade:** o fine-tuning com semente e config fixas reproduz as métricas reportadas.
- **Robustez:** entradas malformadas ou longas demais devem ser tratadas sem crash.

## Marcos Sugeridos

1. **Marco 1 — Dados e tokenização:** Prepare o dataset e um pipeline de tokenização que lida com entradas longas.
2. **Marco 2 — Fine-tuning:** Faça fine-tuning do modelo (completo e LoRA); rastreie e compare execuções.
3. **Marco 3 — Otimizar e servir:** Quantize, agrupe em batch e exponha uma API dentro do orçamento.
4. **Marco 4 — Avaliar e sondar:** Reporte métricas da tarefa e rode uma análise de viés/modos de falha.

## Esboço de Dados e Interface

```text
 texto rotulado
     |
     v
 +--------------------+   trunca/padding, fatia docs longos
 | Tokenizer          |
 +---------+----------+
           v
 +--------------------+   FT completo  OU  adapters LoRA
 | Modelo pré-treinado|   precisão mista, semente, checkpoint(melhor)
 | (BERT/T5/LLM)      |
 +---------+----------+
           v
 +--------------------+   quantização int8/4-bit, batching dinâmico
 | Inferência otimiz. |
 +---------+----------+
           v
   POST /infer { text | texts[] } -> { label|spans|summary, score, model_version }

 Avaliação: métrica da tarefa (F1/ROUGE/acc)
          + sonda de viés (gap de desempenho por grupo)
          + delta de qualidade: fp16 vs quantizado
```

## Desafios Extras

- Adicione augmentation por retrieval para que o modelo fundamente respostas em um repositório de documentos.
- Suporte inferência multi-tarefa atrás de um endpoint via adapters específicos por tarefa.
- Adicione geração em streaming para tarefas de sumarização/geração.
- Adicione uma análise de calibração (diagrama de confiabilidade) para a confiança de classificação.

## Definição de Pronto

- [ ] O modelo tem fine-tuning e supera um zero-shot ou baseline na métrica da tarefa.
- [ ] A tokenização lida com entradas longas demais sem erros silenciosos de truncamento.
- [ ] Uma execução LoRA é comparada com o fine-tuning completo em qualidade e custo.
- [ ] O modelo quantizado e em batch é servido dentro do orçamento de latência/memória declarado.
- [ ] A avaliação inclui métricas da tarefa e ao menos uma sonda de viés/justiça.

## Armadilhas Comuns

- Truncar silenciosamente entradas longas e perder a parte do texto que carregava o sinal.
- Fazer fine-tuning completo em hardware pequeno demais em vez de recorrer a LoRA/quantização.
- Não comparar com nenhuma baseline, sem saber se o fine-tuning realmente ajudou.
- Reportar acurácia agregada enquanto um subgrupo demográfico tem desempenho muito pior.

## Recursos

- [Documentação do Hugging Face Transformers](https://huggingface.co/docs/transformers/index) — fine-tuning, tokenização e inferência.
- [Documentação do Hugging Face PEFT (LoRA)](https://huggingface.co/docs/peft/index) — fine-tuning eficiente em parâmetros.
- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762) — a arquitetura transformer.
- [LoRA: Low-Rank Adaptation of Large Language Models (Hu et al., 2021)](https://arxiv.org/abs/2106.09685) — o método LoRA.
