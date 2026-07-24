# Pipeline de ML (Treinar + Validar + Implantar)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Um notebook que treina um bom modelo é um começo; um pipeline que você pode reexecutar amanhã, sobre dados novos, com um resultado versionado que você consegue servir, é o trabalho de verdade. Este projeto amarra tudo o que um praticante intermediário precisa: um fluxo reproduzível dos dados brutos passando por engenharia de features, treino, validação e um endpoint de predição servido, com o modelo e suas métricas versionados em um registro. A espinha metodológica é um split limpo treino/validação/teste imposto *dentro* do pipeline, para que cada reexecução avalie honestamente e o número que você promove a "produção" seja o medido em dados que o modelo nunca viu.

## Pré-requisitos

- Conforto para treinar um modelo e construir um transformador de features (veja [Pipeline de Engenharia de Features](../06-feature-engineering/))
- Entender a metodologia treino/validação/teste e métricas de avaliação
- Familiaridade com funções/módulos e uma opção de serving (um pequeno framework HTTP)
- Um conjunto rotulado adequado a uma tarefa supervisionada

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Compor carregamento de dados, engenharia de features, treino e avaliação em um pipeline executável
- Impor um split treino/validação/teste sem vazamento como um estágio do pipeline, não um remendo
- Persistir um modelo treinado com suas métricas e metadados em um registro simples
- Servir predições a partir do modelo persistido atrás de um endpoint
- Reexecutar o pipeline inteiro de forma reproduzível e comparar uma nova versão do modelo com a atual

## Requisitos Funcionais

1. O pipeline deve rodar de ponta a ponta dos dados brutos a um modelo avaliado e salvo com um comando.
2. Deve dividir os dados em treino/validação/teste e ajustar todos os transformadores só no treino.
3. Deve avaliar no conjunto de teste retido e registrar as métricas junto ao artefato do modelo.
4. Deve versionar cada modelo (id, timestamp, métricas, hash dos dados) em um registro.
5. Deve carregar uma versão de modelo escolhida e servir predições atrás de um endpoint.
6. Deve validar as entradas de predição e rejeitar requisições malformadas.
7. Deve permitir que um novo modelo seja comparado com o atual antes da promoção.

## Marcos Sugeridos

1. **Marco 1 — Pipeline de treino:** Carregar, dividir, criar features, treinar, avaliar no teste.
2. **Marco 2 — Registro:** Persistir modelo + métricas + metadados e versioná-lo.
3. **Marco 3 — Servir e comparar:** Carregar uma versão, servir predições, comparar candidatos.

## Esboço de Dados e Interface

```text
Entrada do registro
  model_id   : string
  created_at : ISO-8601
  metrics    : { accuracy, f1, auc, ... no TESTE }
  data_hash  : string   (quais dados o produziram)
  path       : local do artefato

Estágios do pipeline
  1. carregar bruto -> validar schema
  2. dividir -> treino / valid / teste  (seed fixa, registrada)
  3. ajustar transformadores de feature no TREINO
  4. treinar modelo; ajustar no VALID
  5. avaliação final no TESTE -> métricas
  6. registrar(modelo, métricas, metadados)

Serving
  POST /predict  body: { features: {...} }
                 -> 200 { prediction, model_id } | 400 inválido
  GET  /models   -> listar versões + métricas
```

## Desafios Extras

- Adicione retreino agendado e só promova um candidato se ele superar o vigente no teste.
- Emita monitoramento básico (latência de predição, distribuição de entrada) para uma checagem de drift consumir.
- Adicione rollback: reaponte o serving para uma versão anterior do modelo por id.
- Containerize o componente de serving e parametrize a versão do modelo que ele carrega.

## Definição de Pronto

- [ ] Um comando roda carregar → dividir → features → treinar → avaliar-no-teste → registrar.
- [ ] Os transformadores são ajustados só nos dados de treino; a métrica de teste é em dados não vistos.
- [ ] Toda versão de modelo carrega suas métricas, timestamp e hash dos dados no registro.
- [ ] O endpoint serve uma versão escolhida e valida suas entradas.
- [ ] Um novo candidato pode ser comparado ao modelo atual antes da promoção.

## Armadilhas Comuns

- Ajustar o scaler/encoder antes do split, inflando a métrica de "teste".
- Servir um modelo sem registrar quais dados ou código o produziram — irreproduzível.
- Pular a validação de entrada, deixando o endpoint quebrar ou prever errado silenciosamente com payloads ruins.
- Promover um novo modelo por um número de validação sem nunca tocar o verdadeiro conjunto de teste.

## Recursos

- [scikit-learn: Pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) — compondo fluxos reproduzíveis.
- [MLflow: Tracking e Model Registry](https://mlflow.org/docs/latest/index.html) — versionando modelos e métricas.
- [Google: entrega contínua de MLOps](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) — níveis de maturidade de pipeline.
- [Documentação do FastAPI](https://fastapi.tiangolo.com/) — uma forma comum de servir predições de modelo.
