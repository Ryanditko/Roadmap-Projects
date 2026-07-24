# Script de Validação de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Antes que os dados possam ser confiados, eles precisam ser verificados. Construa um script que recebe um conjunto de dados e um conjunto de regras — "esta coluna é obrigatória", "isto deve ser um número positivo", "e-mails devem parecer e-mails" — e produz um relatório de exatamente quais linhas quebraram quais regras. Este é o guarda-corpo que fica na porta de entrada de todo pipeline sério: transforma "os dados parecem estranhos" em um diagnóstico preciso, linha a linha. O trabalho de design interessante não são as verificações em si, mas como você expressa regras de forma limpa e como reporta falhas para que um humano consiga de fato agir sobre elas.

## Pré-requisitos

- Capacidade de ler um conjunto de dados CSV ou JSON (veja [Carregador de CSV para Banco de Dados](../01-csv-to-database/) se for novo)
- Conforto com condicionais, laços e manipulação básica de strings
- Familiaridade com expressões regulares em nível iniciante
- Entender tipos de dados: string, número, data, booleano

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Expressar regras de validação como dados ou pequenas funções em vez de blocos `if` emaranhados
- Distinguir verificações estruturais (tipo, obrigatório) de verificações semânticas (faixa, formato)
- Coletar *todas* as violações de uma linha em vez de parar na primeira
- Produzir um relatório legível por humanos e outro legível por máquinas
- Calcular métricas de qualidade como taxas de completude e validade por coluna
- Sair com um código de status ao qual um agendador ou job de CI possa reagir

## Requisitos Funcionais

1. O script deve aceitar um conjunto de dados e um conjunto de regras descrevendo restrições por coluna.
2. Deve suportar ao menos: obrigatório/NOT NULL, verificação de tipo, faixa numérica e formato via regex.
3. Cada linha deve ser verificada contra todas as regras aplicáveis, coletando toda falha, não apenas a primeira.
4. A saída deve incluir um resumo (linhas verificadas, linhas com erros, contagem de erros por regra) e detalhe por linha.
5. Um relatório legível por máquina (JSON ou CSV de violações) deve ser gravado para uso downstream.
6. O processo deve sair com código diferente de zero quando qualquer regra falhar, para que a automação possa barrar a execução.

## Marcos Sugeridos

1. **Marco 1 — Uma regra:** Verifique uma única regra de coluna obrigatória e imprima os números das linhas que falharem.
2. **Marco 2 — Conjunto de regras:** Suporte múltiplos tipos de regra lidos de uma configuração e colete todas as violações por linha.
3. **Marco 3 — Relatório:** Emita um resumo, um arquivo de violações, métricas de qualidade e um código de saída adequado.

## Esboço de Dados e Interface

```text
conjunto de regras (config)
  age    -> {required: true, type: int, min: 0, max: 120}
  email  -> {required: true, regex: "^[^@]+@[^@]+\\.[^@]+$"}
  status -> {allowed: ["active","churned","trial"]}

linha de entrada (linha 7)
  {"age": "-3", "email": "bob@", "status": "active"}

violações coletadas
  linha 7 age   -> abaixo do min (0)
  linha 7 email -> falha no regex

resumo do relatório
  linhas=1000 limpas=944 com_erros=56
  por_regra: age.min=12 email.regex=31 status.allowed=13
  métricas: email.validade=96.9% age.completude=99.1%

código de saída: 1 (há violações)
```

## Desafios Extras

- Adicionar regras entre campos (ex.: `end_date` deve ser posterior a `start_date`).
- Suportar verificações de unicidade em todo o conjunto de dados (detectar chaves duplicadas).
- Adicionar sinalizadores simples de anomalia estatística (valores além de N desvios-padrão).
- Tornar a severidade das regras configurável (erro vs aviso) e falhar apenas em erros.

## Definição de Pronto

- [ ] As regras são definidas como configuração, não codificadas por conjunto de dados.
- [ ] Cada linha com falha lista todas as suas violações, não apenas uma.
- [ ] Tanto um resumo humano quanto um arquivo de violações legível por máquina são produzidos.
- [ ] Métricas de qualidade por coluna são reportadas.
- [ ] O script sai com código diferente de zero exatamente quando há violações.

## Armadilhas Comuns

- Parar no primeiro erro de uma linha, escondendo os outros problemas do usuário.
- Tratar uma string vazia, `null` e uma chave ausente como a mesma coisa sem decidir intencionalmente.
- Regexes rígidas demais (rejeitando e-mails válidos) ou frouxas demais (aceitando lixo).
- Reportar apenas contagens sem números de linha, de modo que ninguém consegue achar as linhas ruins.

## Recursos

- [Documentação do Great Expectations](https://docs.greatexpectations.io/docs/home/) — o vocabulário de referência para "expectativas" de validação de dados.
- [Documentação do Pandera](https://pandera.readthedocs.io/en/stable/) — validação de esquema e estatística para dataframes.
- [MDN: Expressões regulares](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Guide/Regular_expressions) — uma introdução prática e sólida a regex.
- [JSON Schema](https://json-schema.org/learn/getting-started-step-by-step) — uma forma padrão de expressar regras de validação estrutural.
