# Painel Administrativo (CRUD)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa a tela de back-office que todo produto acaba precisando: uma tabela de registros que um operador possa ler, criar, editar e excluir contra uma API real. A parte interessante não são os botões — é manter uma tabela de dados honesta enquanto o servidor é a fonte da verdade. Você vai equilibrar ordenação, filtragem e paginação no servidor, manter formulários e validação em sincronia com a tabela, proteger ações destrutivas atrás de confirmação e refletir o papel de um usuário no que ele tem permissão para tocar. É um tour compacto pelas preocupações de estado, busca de dados e formulários que dominam ferramentas internas reais.

## Pré-requisitos

- Conforto para construir componentes e gerenciar estado local em um framework de componentes (React, Vue, Svelte ou Angular)
- Buscar dados de uma API REST e tratar promessas/async
- Entendimento básico de inputs de formulário controlados e validação no cliente
- Familiaridade com query strings e como elas carregam estado (`?page=2&sort=name`)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Alimentar uma tabela a partir de estado de servidor (ordenar, filtrar, paginar) em vez de carregar tudo de uma vez
- Modelar carregamento, vazio, erro e sucesso como estados de UI explícitos em vez de algo secundário
- Construir formulários de criação/edição que validam antes de enviar e exibem erros em nível de campo
- Proteger operações destrutivas com confirmação e feedback otimista ou pendente
- Refletir o papel de um usuário na interface, desabilitando ou ocultando ações que o papel não pode executar
- Manter o estado da tabela compartilhável e restaurável via URL

## Requisitos Funcionais

1. A tabela deve listar registros com colunas e suportar ordenação por pelo menos duas colunas via servidor.
2. A filtragem (por uma query de texto e ao menos uma faceta de campo) deve re-consultar o servidor, não filtrar um array local obsoleto.
3. A paginação deve requisitar uma página por vez e mostrar a contagem total / o intervalo atual.
4. Uma ação "Novo" deve abrir um formulário que valida campos obrigatórios e formatos antes de permitir o envio.
5. Editar um registro existente deve pré-preencher o formulário e persistir as mudanças via API.
6. A exclusão deve exigir uma etapa de confirmação explícita e não pode disparar em um único clique acidental.
7. Toda ação assíncrona deve renderizar um estado de carregamento e um estado de erro recuperável (com nova tentativa).
8. Ações para as quais o papel atual não tem permissão devem ser desabilitadas ou ocultas, com um rótulo acessível explicando o porquê.

## Marcos Sugeridos

1. **Marco 1 — Ler:** Busque e renderize a tabela com estados de carregamento/vazio/erro.
2. **Marco 2 — Consultar:** Adicione ordenação, filtragem e paginação orientadas pelo servidor, sincronizadas à URL.
3. **Marco 3 — Escrever:** Adicione formulários de criação e edição com validação e persistência via API.
4. **Marco 4 — Excluir e papéis:** Adicione exclusão confirmada, seleção em massa e ações restritas por papel.

## Esboço de Dados e Interface

```text
Layout
+---------------------------------------------------------+
| Barra: [busca] [papel: admin ▾]        [+ Novo registro]|
+---------------------------------------------------------+
| [ ] Nome ▲    | Email          | Papel  | Status |  ... |
| [x] Ada L.    | ada@example.com     | admin  | ativo  |  ⋯   |
| ...                                                      |
+---------------------------------------------------------+
| Selecionados: 2 [Excluir]   Página 2 de 9 ‹ 21–40 / 174›|
+---------------------------------------------------------+

Formato do estado
  records: Record[]        query: { q, sort, dir, page, pageSize, filters }
  status: 'idle'|'loading'|'error'|'ready'
  selection: Set<id>       editing: Record | null

Contrato de API consumido
  GET  /api/records?q=&sort=name&dir=asc&page=2&pageSize=20
       -> 200 { items: Record[], total: 174 }
  POST /api/records        body: {..}  -> 201 Record | 422 { errors }
  PUT  /api/records/{id}   body: {..}  -> 200 Record | 422 { errors }
  DELETE /api/records/{id}             -> 204 | 403
```

## Desafios Extras

- Adicione operações em massa (excluir/desativar vários) com uma única confirmação resumindo a contagem.
- Persista a visibilidade das colunas e o tamanho da página no local storage por usuário.
- Adicione atualizações otimistas para edições, revertendo em caso de requisição falha.
- Exporte a visão filtrada atual para CSV.

## Definição de Pronto

- [ ] Ordenação, filtragem e paginação fazem round-trip ao servidor e sobrevivem a um refresh via URL.
- [ ] Formulários de criação e edição bloqueiam o envio em input inválido e mostram erros de campo vindos de um 422.
- [ ] A exclusão sempre passa por uma confirmação explícita e nunca dispara em um único clique.
- [ ] Estados de carregamento, vazio e erro são visíveis e o estado de erro oferece nova tentativa.
- [ ] Ações restritas por papel são desabilitadas ou ocultas com uma explicação acessível.

## Armadilhas Comuns

- Buscar todo o conjunto de dados e ordenar/filtrar no cliente — funciona com 20 linhas e morre com 20.000.
- Tratar carregamento e erro como a ausência de dados, deixando o usuário encarando uma tabela vazia.
- Perder o estado da tabela no refresh porque ele vive só na memória, não na URL.
- Disparar uma exclusão no clique da linha sem confirmação, e depois não ter como desfazer.
- Confiar na verificação de papel no cliente para segurança — é uma conveniência de UX; o servidor ainda precisa impor as permissões.

## Recursos

- [MDN: Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API) — fazendo e tratando requisições de API.
- [MDN: Validação de formulário no cliente](https://developer.mozilla.org/pt-BR/docs/Learn/Forms/Form_validation) — validando input da maneira certa.
- [web.dev: Tabelas de dados acessíveis](https://web.dev/articles/grid-role) — semântica e comportamento de teclado para grids.
- [ARIA Authoring Practices: Padrão Grid](https://www.w3.org/WAI/ARIA/apg/patterns/grid/) — o contrato de interação para tabelas editáveis.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — onde essas habilidades se situam no quadro geral.
