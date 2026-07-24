# API de Notas com Persistência em Arquivo

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 5–8 horas

## Visão Geral

Construa uma API de notas cujos dados sobrevivem a um reinício gravando no sistema de arquivos em vez de manter tudo em memória. Este é o passo natural depois do CRUD em memória: os mesmos endpoints, mas agora toda alteração precisa ser refletida em disco, e o servidor deve recarregar as notas existentes ao iniciar. Você vai encarar as questões reais da persistência — serialização, escritas atômicas e o que acontece quando duas escritas competem — na menor escala possível.

## Pré-requisitos

- Experiência construindo endpoints CRUD ([API REST Simples de Tarefas](../01-simple-rest-api-task-management/) é a versão em memória disto)
- Entendimento de serialização e desserialização JSON
- Familiaridade com a E/S de arquivos e o tratamento de erros da sua linguagem
- Consciência do diretório de trabalho e de caminhos relativos vs absolutos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Ler e escrever dados estruturados (JSON) no sistema de arquivos com segurança
- Carregar o estado persistido em memória na inicialização e mantê-lo sincronizado
- Realizar uma escrita atômica para que uma falha no meio do salvamento não corrompa o armazenamento
- Tratar erros de sistema de arquivos (arquivo ausente, permissão negada) sem quebrar
- Atribuir IDs únicos e estáveis que sobrevivem a reinícios

## Requisitos Funcionais

1. O sistema deve suportar criar, ler, atualizar e remover notas via HTTP.
2. Toda mutação deve ser persistida para que os dados sobrevivam a um reinício do servidor.
3. Na inicialização, o sistema deve carregar as notas existentes do disco, ou começar vazio se não houver nenhuma.
4. Uma nota deve ter um ID único que não colida após reinícios.
5. Ler, atualizar ou remover uma nota inexistente deve retornar 404.
6. O sistema não deve corromper o arquivo de dados se uma escrita falhar ou o processo for morto durante o salvamento.
7. O sistema deve retornar um erro claro (não uma falha) quando o arquivo de armazenamento estiver ilegível.

## Marcos Sugeridos

1. **Marco 1 — Carregar e listar:** Leia as notas de um arquivo JSON na inicialização e sirva-as; crie o arquivo se ausente.
2. **Marco 2 — Persistir escritas:** Faça criar/atualizar/remover gravar as mudanças de volta no disco.
3. **Marco 3 — Segurança:** Adicione escritas atômicas (escrever-temp-e-renomear) e tratamento gracioso de erros de E/S.

## Esboço de Dados e Interface

```text
Arquivo de armazenamento: notes.json
[
  { "id": "n_01", "title": "...", "body": "...", "updatedAt": "ISO-8601" }
]

GET    /notes            -> 200 [Note, ...]
GET    /notes/{id}       -> 200 Note | 404
POST   /notes            -> 201 Note   body: { title, body }
PUT    /notes/{id}       -> 200 Note | 404
DELETE /notes/{id}       -> 204 | 404

Escrita atômica: escrever notes.json.tmp -> fsync -> renomear sobre notes.json
```

## Desafios Extras

- Adicione busca por título e filtro por intervalo de datas de `updatedAt`.
- Armazene cada nota em seu próprio arquivo em vez de um único índice e compare os trade-offs.
- Aplique debounce nas escritas para que edições rápidas se agrupem em menos operações de disco.
- Adicione uma cópia de backup com timestamp antes de cada sobrescrita.

## Definição de Pronto

- [ ] Notas criadas antes de um reinício estão presentes depois dele.
- [ ] O servidor inicia de forma limpa exista ou não o arquivo de armazenamento.
- [ ] Matar o processo no meio da escrita deixa o arquivo válido anterior intacto, não um truncado.
- [ ] IDs inexistentes retornam 404 e arquivos ilegíveis retornam 500 com mensagem clara, não um stack trace para o cliente.
- [ ] Os IDs permanecem únicos ao longo de múltiplos reinícios.

## Armadilhas Comuns

- Sobrescrever o arquivo no lugar com um buffer parcial, corrompendo todas as notas quando uma escrita falha — use escrever-e-renomear.
- Derivar IDs do tamanho do array, o que se repete após remoções e entre reinícios.
- Esquecer de criar o arquivo de armazenamento (ou seu diretório) na primeira execução, causando falha na inicialização.
- Manter o arquivo inteiro aberto ou lê-lo a cada requisição em vez de manter uma cópia em memória sincronizada com o disco.

## Recursos

- [Node.js: módulo fs](https://nodejs.org/api/fs.html) — referência de E/S de arquivos se você usar Node.
- [Python: Lendo e Escrevendo Arquivos](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files) — a abordagem padrão do Python.
- [Wikipedia: Atomicidade (write-rename)](https://en.wikipedia.org/wiki/Atomicity_(database_systems)) — por que renomear é o truque de escrita segura.
- [JSON.org](https://www.json.org/json-pt.html) — o formato de dados JSON em resumo.
