# Sistema de Predição em Tempo Real

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um sistema de serving de modelos que responde a requisições de predição em alto volume com latência baixa e previsível. Qualquer um envolve um modelo em uma rota Flask; a parte difícil é manter a latência p99 dentro do orçamento enquanto as requisições chegam em rajadas, as features precisam ser buscadas ou calculadas na hora e o próprio modelo pode ser grande. Este projeto te empurra para a caixa de ferramentas real de serving: batching de requisições, otimização de modelo (quantização, export ONNX), um cache quente de features e degradação graciosa quando uma dependência fica lenta. Você vai medir tudo, porque "rápido o suficiente" só faz sentido diante de números.

## Pré-requisitos

- Um modelo treinado que você possa exportar (scikit-learn, PyTorch ou TensorFlow)
- Domínio sólido de serviços HTTP, concorrência e I/O assíncrono
- Familiaridade com um cache (Redis) e teste de carga básico (Locust, k6 ou wrk)
- Conforto para ler percentis de latência, não apenas médias

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Otimizar um modelo para inferência via quantização, pruning ou export ONNX/TensorRT
- Implementar batching dinâmico de requisições para elevar a vazão sem destruir a latência
- Projetar um cache de features e raciocinar sobre staleness vs frescor
- Aplicar backpressure, timeouts e circuit breakers para que a sobrecarga falhe de forma limpa
- Medir e defender latência p50/p95/p99 e vazão sob carga

## Requisitos Funcionais

1. O sistema deve servir predições por uma API com schema de requisição/resposta documentado.
2. Requisições recebidas devem ser agrupadas dinamicamente até um tamanho/janela de tempo antes da inferência.
3. Features usadas com frequência devem ser servidas de um cache com TTL explícito e caminho de miss.
4. O sistema deve impor timeouts por requisição e descartar ou enfileirar carga quando saturado.
5. Um fallback (predição em cache, padrão ou modelo mais leve) deve entrar em ação quando o caminho primário falha.
6. O sistema deve expor métricas de latência e vazão por endpoint.
7. A acurácia do modelo otimizado deve ser validada contra a baseline não otimizada dentro de uma tolerância declarada.

## Requisitos Não Funcionais

- **Latência:** p95 ≤ um orçamento declarado (ex.: 50 ms) e p99 limitado, sob a taxa de requisições alvo.
- **Vazão:** sustentar um RPS definido (ex.: 1.000) no hardware alvo.
- **Disponibilidade:** degradar em vez de quebrar sob sobrecarga; sem filas ilimitadas.
- **Consistência:** o staleness do cache de features deve ser limitado e documentado.

## Marcos Sugeridos

1. **Marco 1 — Serving baseline:** Exponha o modelo atrás de uma API e meça latência/vazão baseline.
2. **Marco 2 — Otimizar o modelo:** Exporte para ONNX e/ou quantize; verifique o delta de acurácia e o ganho de velocidade.
3. **Marco 3 — Batching e cache:** Adicione batching dinâmico e um cache de features; meça de novo.
4. **Marco 4 — Resiliência:** Adicione timeouts, circuit breakers e um fallback; faça teste de carga até a saturação.

## Esboço de Dados e Interface

```text
 cliente --> POST /predict {features|entity_id}
                 |
                 v
        +-----------------+   miss cache   +--------------+
        | Busca features  |--------------->| Feature store |
        | (cache Redis)   |<---------------|  / BD         |
        +--------+--------+                +--------------+
                 |
                 v
        +-----------------+   janela: N reqs ou T ms
        | Fila de batching|------------------------+
        +--------+--------+                        |
                 v                                 v
        +-----------------+                +----------------+
        | Modelo otimizado| -- falha ----> | Caminho fallback|
        | (ONNX/quantizado)|               | cache/padrão   |
        +--------+--------+                +----------------+
                 v
   resposta { prediction, model_version, latency_ms }

Métricas: latência p50/p95/p99, RPS, tamanho de batch, taxa de acerto do cache
```

## Desafios Extras

- Adicione um caminho GPU com TensorRT e compare custo/latência com CPU.
- Suporte serving A/B ou shadow de duas versões de modelo com métricas por versão.
- Adicione batching adaptativo que ajusta a janela conforme a carga ao vivo.
- Pré-calcule e aqueça o cache para as entidades mais quentes na inicialização.

## Definição de Pronto

- [ ] Latência p95 e p99 são medidas sob carga alvo e atendem ao orçamento declarado.
- [ ] O delta de acurácia do modelo otimizado versus baseline está documentado e é aceitável.
- [ ] Batching dinâmico e o cache de features estão no lugar com métricas visíveis de taxa de acerto.
- [ ] A sobrecarga dispara backpressure e o fallback, nunca uma fila ilimitada ou crash.
- [ ] Um relatório de teste de carga mostra o comportamento da carga normal até a saturação.

## Armadilhas Comuns

- Otimizar a latência de uma requisição única e nunca testar sob carga concorrente.
- Fazer batching tão agressivo que a latência de cauda explode para a última requisição de cada janela.
- Cachear features sem TTL, fazendo o modelo servir silenciosamente entradas obsoletas.
- Reportar latência média e esconder um p99 terrível atrás dela.

## Recursos

- [Documentação do ONNX Runtime](https://onnxruntime.ai/docs/) — otimização de inferência entre frameworks.
- [NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/index.html) — batching dinâmico e serving multi-modelo.
- [Arquitetura do TensorFlow Serving](https://www.tensorflow.org/tfx/serving/architecture) — padrões de serving em produção.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — backpressure e degradação graciosa.
