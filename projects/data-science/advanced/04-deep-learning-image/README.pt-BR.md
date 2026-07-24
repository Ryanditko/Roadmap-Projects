# Classificador de Imagens com Deep Learning

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Treine um classificador de imagens convolucional com qualidade de produção em um dataset real e depois otimize-o para deployment. A modelagem é só metade do projeto: a parte avançada é fazer transfer learning corretamente (congelar e depois fazer fine-tuning com learning rates discriminativas), construir um pipeline de augmentation que ajuda em vez de atrapalhar e então encolher a rede treinada para que sirva dentro de um orçamento de memória e latência. Você vai terminar com um modelo, um pipeline de treino reproduzível e um artefato otimizado cuja perda de acurácia você consegue quantificar.

## Pré-requisitos

- Conhecimento prático de redes neurais e backpropagation
- Experiência com PyTorch ou TensorFlow/Keras e treino em GPU
- Familiaridade com um dataset de imagens rotulado (CIFAR-100, Food-101 ou o seu próprio)
- Entendimento de overfitting, regularização e schedules de learning rate

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Aplicar transfer learning: congelar um backbone e depois fazer fine-tuning com learning rates por camada
- Construir um pipeline de augmentation e medir seu efeito na acurácia de validação
- Usar scheduling de learning rate, early stopping e checkpointing em um loop de treino real
- Otimizar um modelo treinado via quantização, pruning ou export (ONNX/TFLite)
- Quantificar o tradeoff acurácia-vs-tamanho-vs-latência do modelo otimizado

## Requisitos Funcionais

1. O pipeline deve carregar um dataset de imagens rotulado com splits treino/val/teste reproduzíveis.
2. O treino deve partir de um backbone pré-treinado (ResNet, EfficientNet ou MobileNet) e fazer fine-tuning.
3. Um pipeline de augmentation deve ser configurável e aplicado apenas ao split de treino.
4. O treino deve fazer checkpoint do melhor modelo pela métrica de validação e suportar retomada.
5. O sistema deve reportar métricas por classe e uma matriz de confusão no split de teste.
6. O modelo treinado deve ser exportado e otimizado para inferência (quantizado e/ou ONNX/TFLite).
7. A acurácia do modelo otimizado deve ser medida contra a baseline em precisão total.

## Requisitos Não Funcionais

- **Reprodutibilidade:** sementes, splits e config de augmentation devem reproduzir as métricas reportadas.
- **Orçamento de deployment:** o modelo otimizado deve atender a um tamanho declarado (ex.: ≤ 25 MB) e a uma meta de latência em CPU.
- **Tolerância de acurácia:** a queda de acurácia induzida pela otimização deve ficar dentro de um limite documentado.
- **Vazão:** a vazão de inferência em batch no hardware alvo deve ser reportada.

## Marcos Sugeridos

1. **Marco 1 — Dados e baseline:** Splits reproduzíveis, um data loader e uma baseline do zero ou com backbone congelado.
2. **Marco 2 — Transfer learning:** Faça fine-tuning com learning rates discriminativas e augmentation; supere a baseline.
3. **Marco 3 — Avaliação:** Métricas por classe, matriz de confusão e análise de erros no split de teste.
4. **Marco 4 — Otimizar e exportar:** Quantize/pode, exporte e quantifique o tradeoff acurácia/tamanho/latência.

## Esboço de Dados e Interface

```text
 dataset/
   train/ val/ test/   (split estratificado, com semente)
        |
        v
 +---------------------+   só treino: flips, crop, color jitter, mixup
 | Augmentation        |
 +----------+----------+
            v
 +---------------------+   backbone congelado -> depois fine-tune
 | CNN pré-treinada    |   ResNet/EfficientNet/MobileNet
 |  + nova head        |   schedule LR, early stop, checkpoint(melhor)
 +----------+----------+
            v
 +---------------------+
 | Avaliação           |   P/R/F1 por classe, matriz de confusão
 +----------+----------+
            v
 +---------------------+   quantizar / podar / exportar
 | Artefato otimizado  |   ONNX / TFLite
 +---------------------+
   relatório: {acc_fp32, acc_int8, size_mb, cpu_latency_ms}
```

## Desafios Extras

- Adicione visualizações Grad-CAM para inspecionar onde o modelo presta atenção.
- Trate desbalanceamento de classes com loss ponderada ou focal loss e meça o efeito.
- Adicione test-time augmentation e compare acurácia vs custo de latência.
- Destile o modelo com fine-tuning em uma rede aluno menor.

## Definição de Pronto

- [ ] Splits, sementes e config de augmentation reproduzem as métricas reportadas.
- [ ] O modelo de transfer learning com fine-tuning supera a baseline no split de teste.
- [ ] Métricas por classe e uma matriz de confusão são produzidas e brevemente analisadas.
- [ ] Um artefato otimizado é exportado e atende ao orçamento de tamanho/latência declarado.
- [ ] O delta de acurácia entre os modelos em precisão total e otimizado está documentado.

## Armadilhas Comuns

- Aplicar augmentation aos splits de validação/teste e inflar a robustez aparente.
- Fazer fine-tuning de toda a rede com learning rate alta e destruir os pesos pré-treinados.
- Reportar apenas acurácia top-1 enquanto uma classe minoritária vai silenciosamente muito mal.
- Esquecer de remedir a acurácia após a quantização e enviar um modelo degradado.

## Recursos

- [Tutorial de Transfer Learning do PyTorch](https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html) — congelamento e fine-tuning feitos direito.
- [TensorFlow: Data augmentation](https://www.tensorflow.org/tutorials/images/data_augmentation) — construindo um pipeline de augmentation.
- [Documentação de Quantização do PyTorch](https://pytorch.org/docs/stable/quantization.html) — abordagens post-training e quantization-aware.
- [Deep Residual Learning for Image Recognition (He et al., 2015)](https://arxiv.org/abs/1512.03385) — o paper da ResNet por trás da maioria dos backbones modernos.
