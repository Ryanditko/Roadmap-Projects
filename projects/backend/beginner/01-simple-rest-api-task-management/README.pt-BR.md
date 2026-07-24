# API REST Simples para Gerenciamento de Tarefas

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 4–7 horas

## Visão Geral

Construa uma API REST que gerencia uma lista de tarefas inteiramente em memória, sem banco de dados. Imagine o backend por trás de um aplicativo de tarefas simples: o servidor mantém as tarefas em uma lista enquanto está no ar, e os clientes as criam, leem, atualizam e removem via HTTP. Este é o primeiro projeto de backend clássico porque isola a habilidade central — mapear verbos HTTP para operações sobre um recurso — sem a distração da persistência.

## Pré-requisitos

- Entendimento básico de HTTP: o que são requisição, resposta, método e código de status
- Familiaridade com o gerenciador de pacotes da sua linguagem e como rodar um processo local
- Um framework web à sua escolha (Express no Node, Flask/FastAPI no Python, Gin no Go)
- Uma ferramenta para enviar requisições: `curl`, HTTPie, Postman ou um cliente REST do editor

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Mapear as operações CRUD para os métodos HTTP corretos (GET, POST, PUT/PATCH, DELETE)
- Escolher códigos de status significativos (200, 201, 204, 400, 404) para cada resultado
- Analisar um corpo de requisição JSON e serializar uma resposta JSON
- Validar dados recebidos antes de alterar seu armazenamento em memória
- Gerar identificadores únicos e estáveis para novos recursos
- Explicar por que o estado em memória desaparece ao reiniciar e quando isso é aceitável

## Requisitos Funcionais

1. O sistema deve expor um endpoint para listar todas as tarefas.
2. O sistema deve expor um endpoint para buscar uma tarefa pelo ID e retornar 404 quando ela não existir.
3. O sistema deve criar uma tarefa a partir de um corpo JSON, atribuir um ID único e retornar 201 com o recurso criado.
4. O sistema deve rejeitar requisições de criação sem um campo obrigatório com 400 e uma mensagem de erro útil.
5. O sistema deve atualizar uma tarefa existente e retornar 404 se o ID for desconhecido.
6. O sistema deve remover uma tarefa pelo ID e retornar 204 (ou 404 se ausente).
7. O sistema deve retornar JSON válido com o cabeçalho `Content-Type` correto em toda resposta.

## Marcos Sugeridos

1. **Marco 1 — Caminho de leitura:** Sirva um array fixo de tarefas via GET (lista) e GET por ID, incluindo o caso 404.
2. **Marco 2 — Caminho de escrita:** Adicione POST com geração de ID e DELETE, alterando a lista em memória.
3. **Marco 3 — Robustez:** Adicione PUT/PATCH, validação de entrada com respostas 400 e formatos de erro consistentes.

## Esboço de Dados e Interface

```text
Task
  id:        string | number   (atribuído pelo servidor)
  title:     string            (obrigatório)
  completed: boolean           (padrão false)
  createdAt: string ISO-8601

GET    /tasks            -> 200 [Task, ...]
GET    /tasks/{id}       -> 200 Task | 404
POST   /tasks            -> 201 Task    body: { title }
PUT    /tasks/{id}       -> 200 Task | 404
DELETE /tasks/{id}       -> 204 | 404

Formato de erro: { "error": "title is required" }
```

## Desafios Extras

- Adicione um filtro `?completed=true` no endpoint de listagem.
- Suporte PATCH para atualizações parciais além do PUT completo.
- Adicione paginação com parâmetros de query `limit` e `offset`.
- Escreva testes automatizados que exercitem cada endpoint e código de status.

## Definição de Pronto

- [ ] Todos os cinco endpoints CRUD funcionam e retornam os códigos de status documentados.
- [ ] Buscar, atualizar ou remover um ID inexistente retorna 404, não uma falha.
- [ ] Entrada inválida retorna 400 com mensagem clara, nunca um 500.
- [ ] Os IDs são únicos mesmo quando tarefas são criadas em rápida sucessão.
- [ ] Toda resposta define `Content-Type: application/json`.

## Armadilhas Comuns

- Retornar 200 para um recurso criado em vez de 201, ou 200 para um recurso ausente em vez de 404 — revisores percebem isso na hora.
- Usar o índice do array como ID, o que quebra depois que uma remoção desloca a lista.
- Esquecer de analisar o corpo JSON, deixando seu handler com campos `undefined`/`None`.
- Deixar uma exceção não tratada vazar um 500 quando um 400/404 limpo seria a resposta correta.

## Recursos

- [MDN: Métodos de requisição HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods) — a referência canônica do significado de cada verbo.
- [MDN: Códigos de status de resposta HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status) — escolha o código certo para cada resultado.
- [REST API Tutorial](https://restfulapi.net/) — convenções práticas para projetar endpoints de recursos.
- [roadmap.sh: Backend Developer](https://roadmap.sh/backend) — onde esta habilidade se encaixa no panorama geral.
