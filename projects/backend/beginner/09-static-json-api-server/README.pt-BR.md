# Servidor de API JSON Estático

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 3–5 horas

## Visão Geral

Construa um servidor de API somente leitura que carrega arquivos de dados JSON na inicialização e os serve via HTTP com filtragem, paginação e ordenação — um backend mock leve, muito parecido com o `json-server`, contra o qual um frontend pode se desenvolver antes de a API real existir. Esta é a introdução mais suave possível ao trabalho de backend: sem escritas, sem banco de dados, sem auth. Todo o foco está em configurar o servidor, roteamento e moldar resultados de consulta, tornando-o um primeiro projeto ideal ou uma base sobre a qual os outros briefs se apoiam.

## Pré-requisitos

- Programação básica e a capacidade de rodar um processo local
- Entender o que é JSON
- Um framework HTTP mínimo ou o servidor HTTP embutido da sua linguagem
- Familiaridade com query strings de URL (`?chave=valor`)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Subir um servidor HTTP e definir rotas para múltiplos recursos
- Carregar e cachear dados de arquivos JSON na inicialização
- Retornar respostas com o `Content-Type` e códigos de status corretos
- Implementar recursos por parâmetro de query: filtragem, ordenação e paginação
- Tratar rotas desconhecidas e resultados vazios de forma limpa

## Requisitos Funcionais

1. O sistema deve carregar um ou mais arquivos de dados JSON quando inicia.
2. O sistema deve expor um endpoint de listagem por recurso retornando os dados como JSON.
3. O sistema deve expor um endpoint por id retornando um único registro ou 404.
4. O endpoint de listagem deve suportar paginação via parâmetros `limit` e `offset` (ou página).
5. O endpoint de listagem deve suportar filtragem por ao menos um campo via parâmetro de query.
6. O endpoint de listagem deve suportar ordenação por um campo, ascendente e descendente.
7. Rotas desconhecidas devem retornar 404 e toda resposta deve definir `Content-Type: application/json`.

## Marcos Sugeridos

1. **Marco 1 — Servir dados:** Carregue um arquivo JSON na inicialização e sirva a lista completa e uma busca por id.
2. **Marco 2 — Recursos de query:** Adicione paginação e filtragem por campo ao endpoint de listagem.
3. **Marco 3 — Acabamento:** Adicione ordenação, um health check e tratamento consistente de 404.

## Esboço de Dados e Interface

```text
Arquivos de dados carregados no boot: products.json, users.json, ...

GET /products                      -> 200 [ ... ]
GET /products/{id}                 -> 200 { ... } | 404
GET /products?category=books       -> filtrar por campo
GET /products?_sort=price&_order=desc
GET /products?_limit=10&_offset=20 -> paginação
GET /health                        -> 200 { "status": "ok" }

Envelope de resposta (opcional):
  { "data": [ ... ], "total": 128, "limit": 10, "offset": 20 }
```

## Desafios Extras

- Adicione um parâmetro de busca textual `q` em campos selecionados.
- Adicione compressão gzip e cabeçalhos de cache (`ETag` ou `Cache-Control`).
- Adicione cabeçalhos CORS para que um frontend no navegador possa consumir a API.
- Sirva um índice de endpoints autogerado descrevendo os recursos disponíveis.

## Definição de Pronto

- [ ] Os dados carregam uma vez na inicialização e são servidos sem reler arquivos a cada requisição.
- [ ] A listagem suporta paginação, filtragem e ordenação, combináveis em uma requisição.
- [ ] Uma busca por id retorna o registro ou um 404 limpo.
- [ ] Rotas desconhecidas retornam 404, nunca uma falha ou uma página de erro HTML.
- [ ] Toda resposta declara `Content-Type: application/json`.

## Armadilhas Comuns

- Ler e analisar o arquivo JSON a cada requisição em vez de cacheá-lo na inicialização.
- Aplicar ordenação antes do filtro (ou vice-versa) de forma inconsistente, produzindo resultados paginados confusos.
- Erros de fronteira (off-by-one) na paginação, descartando ou duplicando um registro nas viradas de página.
- Retornar um corpo vazio para nenhum resultado em vez de um array vazio `[]`.

## Recursos

- [MDN: Visão geral do HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Overview) — os fundamentos de requisição/resposta.
- [json-server](https://github.com/typicode/json-server) — uma implementação de referência exatamente desta ideia.
- [MDN: URLSearchParams](https://developer.mozilla.org/pt-BR/docs/Web/API/URLSearchParams) — analisando query strings de forma limpa.
- [REST API Tutorial: filtragem e paginação](https://restfulapi.net/) — convenções para recursos baseados em query.
