# Gerenciador de Usuários via CLI

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 4–7 horas

## Visão Geral

Construa uma ferramenta de linha de comando que gerencia uma lista de usuários — adicionar, listar, atualizar, remover e buscar — persistindo-os em um arquivo local. Não há servidor web aqui; a interface é o terminal. Este projeto ensina a outra metade do trabalho de backend: analisar argumentos, dar feedback claro por meio de códigos de saída e saída de texto, e tratar um arquivo simples como um pequeno e confiável armazenamento de dados. É o tipo de ferramenta interna que engenheiros de backend escrevem o tempo todo.

## Pré-requisitos

- Programação básica na linguagem escolhida e como rodar um script pelo terminal
- Entendimento de JSON ou CSV como formato de armazenamento
- Familiaridade em ler e escrever arquivos
- Uma biblioteca de parsing de argumentos (argparse/Click/Typer, Commander, cobra) ou a disposição de analisar o `argv` você mesmo

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma CLI baseada em subcomandos (`tool add`, `tool list`, ...) com flags e argumentos
- Validar entrada e reportar erros com códigos de saída significativos e diferentes de zero
- Persistir registros estruturados em um arquivo e lê-los de volta de forma confiável
- Impor uma restrição de unicidade (ex.: e-mail único) entre os registros
- Formatar a saída do terminal para que seja legível tanto por humanos quanto por scripts

## Requisitos Funcionais

1. A ferramenta deve suportar adicionar um usuário com ao menos nome e e-mail.
2. A ferramenta deve rejeitar um usuário cujo e-mail seja malformado ou já exista, com mensagem clara e saída diferente de zero.
3. A ferramenta deve listar todos os usuários em um formato legível.
4. A ferramenta deve atualizar e remover um usuário identificado por uma chave estável (ID ou e-mail).
5. A ferramenta deve buscar ou filtrar usuários por um campo (trecho de nome ou e-mail).
6. Todas as mudanças devem persistir em um arquivo e estar visíveis na próxima invocação.
7. A ferramenta deve sair com 0 em sucesso e diferente de zero em qualquer erro tratado.

## Marcos Sugeridos

1. **Marco 1 — Adicionar e listar:** Analise um comando `add`, adicione ao arquivo e implemente o `list`.
2. **Marco 2 — Atualizar, remover, buscar:** Complete os verbos CRUD mais um comando de busca/filtro.
3. **Marco 3 — Validação e UX:** Imponha formato e unicidade de e-mail, adicione um `--help` útil e use códigos de saída corretos.

## Esboço de Dados e Interface

```text
Armazenamento: users.json  ->  [ { id, name, email, role, createdAt } ]

tool add    --name "Ada" --email ada@x.com [--role user]
tool list   [--role admin] [--sort name]
tool update --id u_01 --name "Ada L."
tool delete --id u_01
tool search --email ada

Códigos de saída: 0 ok | 1 erro de validação | 2 não encontrado | 3 erro de armazenamento
```

## Desafios Extras

- Adicione um modo interativo com prompts quando flags obrigatórias forem omitidas.
- Adicione saída colorida e um layout de tabela para o `list`.
- Suporte exportar para CSV e importar de volta.
- Adicione uma flag `--json` para que a saída seja legível por máquina para uso em pipes.

## Definição de Pronto

- [ ] Todo subcomando funciona e persiste as mudanças entre invocações separadas.
- [ ] Adicionar um e-mail duplicado ou inválido falha claramente com código de saída diferente de zero.
- [ ] O `--help` descreve todo comando e flag.
- [ ] Remover ou atualizar um usuário inexistente retorna um erro de não-encontrado, não uma falha.
- [ ] Sucesso e falha mapeiam para códigos de saída corretos e documentados.

## Armadilhas Comuns

- Imprimir erros no stdout em vez do stderr, o que quebra pipes e scripts.
- Sair sempre com 0, de modo que quem chama não consegue distinguir sucesso de falha.
- Reescrever o arquivo inteiro de forma não atômica e perder dados se o processo for interrompido.
- Tratar o índice do array como ID do usuário, que se desloca após remoções.

## Recursos

- [Command Line Interface Guidelines](https://clig.dev/) — princípios modernos e práticos de design de CLI.
- [Tutorial de argparse do Python](https://docs.python.org/3/howto/argparse.html) — a abordagem da biblioteca padrão para Python.
- [Wikipedia: Exit status](https://en.wikipedia.org/wiki/Exit_status) — a convenção por trás dos códigos de saída.
- [The Art of Command Line](https://github.com/jlevy/the-art-of-command-line) — fluência mais ampla no terminal.
