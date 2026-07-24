# Exportador de API para CSV

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Uma enorme quantidade de dados vive atrás de APIs HTTP, uma página por vez. Construa uma ferramenta que chama uma API REST paginada, percorre todas as páginas, achata cada registro JSON em uma linha plana e escreve o resultado inteiro em um CSV limpo. As lições reais se escondem nos cantos: seguir a paginação corretamente para obter *todos* os dados e nenhuma duplicata, recuar quando a API te limita por taxa, e repetir uma requisição instável sem corromper sua saída. É o primeiro projeto em que seu programa depende de um sistema que você não controla, então lidar com as falhas dele com elegância é todo o objetivo.

## Pré-requisitos

- Capacidade de fazer uma requisição HTTP GET na linguagem de sua escolha
- Entender a estrutura JSON (objetos, arrays, aninhamento)
- Conforto para escrever um arquivo CSV (veja [Carregador de CSV para Banco de Dados](../01-csv-to-database/) para o básico de CSV)
- Consciência dos códigos de status HTTP, especialmente 429 e 5xx

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Consumir uma API paginada usando paginação por cursor ou por offset
- Respeitar limites de taxa e recuar em `429 Too Many Requests`
- Repetir falhas transitórias com backoff exponencial e um teto
- Achatar JSON aninhado em colunas para um CSV tabular
- Mapear campos da API para cabeçalhos de CSV via configuração, não codificação fixa
- Deduplicar registros por uma chave estável entre páginas

## Requisitos Funcionais

1. A ferramenta deve buscar todas as páginas de um endpoint paginado até não haver mais.
2. Deve tratar respostas `429` e `5xx` repetindo com backoff, até um limite configurável.
3. Deve transformar cada registro JSON em uma linha plana usando um mapeamento de campos definido.
4. Campos aninhados devem ser achatados ou extraídos; arrays devem ter um tratamento documentado (juntar, primeiro ou contar).
5. Registros duplicados (a mesma chave aparecendo em páginas diferentes) devem ser escritos apenas uma vez.
6. A saída deve ser um CSV válido com linha de cabeçalho e um resumo da execução com páginas buscadas e linhas escritas.

## Marcos Sugeridos

1. **Marco 1 — Buscar uma página:** Chame o endpoint uma vez e imprima os registros analisados.
2. **Marco 2 — Paginar:** Siga a paginação até o fim, acumulando registros com retry/backoff.
3. **Marco 3 — Transformar e exportar:** Achate os registros, deduplique e escreva o CSV com um resumo.

## Esboço de Dados e Interface

```text
GET /api/v1/users?page=1  -> { "data": [ ... ], "next": "?page=2" }

registro da api (aninhado)
  { "id": 7, "name": "Ana",
    "address": { "city": "Recife" },
    "tags": ["vip","beta"] }

mapeamento de campos (config)
  id            -> id
  name          -> name
  address.city  -> city         (achata aninhado)
  tags          -> tags         (junta com ";")

linha csv
  id,name,city,tags
  7,Ana,Recife,vip;beta

política de retry
  429/5xx -> espera (2^tentativa * base), máx 5 tentativas, honra Retry-After

resumo: páginas=12 linhas=1180 duplicatas_puladas=4 retries=3
```

## Desafios Extras

- Suportar exportação incremental usando um parâmetro `updated_since` e uma marca d'água salva.
- Transmitir linhas para o CSV conforme as páginas chegam em vez de armazenar tudo em memória.
- Adicionar um cabeçalho de autenticação orientado por configuração (chave de API ou bearer token) lido do ambiente.
- Emitir tanto CSV quanto JSON delimitado por linhas na mesma execução.

## Definição de Pronto

- [ ] Toda página é buscada; a contagem final de linhas bate com o total reportado pela API.
- [ ] Erros de limite de taxa e de servidor disparam backoff e resultam em sucesso eventual ou falha clara.
- [ ] Campos aninhados são achatados corretamente e arrays tratados conforme a regra documentada.
- [ ] Nenhuma chave duplicada aparece no CSV de saída.
- [ ] Um resumo reporta páginas buscadas, linhas escritas e retries.

## Armadilhas Comuns

- Assumir um número fixo de páginas em vez de seguir o sinal "next" da API.
- Martelar a API e ser limitado porque você ignora `429` e `Retry-After`.
- Perder dados aninhados escrevendo um dicionário ou blob JSON direto em uma célula do CSV.
- Armazenar todos os registros em memória para um conjunto grande em vez de transmitir para o disco.

## Recursos

- [MDN: códigos de status de resposta HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status) — o que 429, 500 e 503 significam para a lógica de retry.
- [Google Cloud: Retry com backoff exponencial](https://cloud.google.com/storage/docs/retry-strategy) — uma estratégia de backoff clara e padrão.
- [Python `requests`](https://requests.readthedocs.io/en/latest/) — o cliente HTTP ergonômico para a camada de busca.
- [Padrões de paginação de API REST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Link) — como as APIs sinalizam a próxima página via cabeçalhos.
