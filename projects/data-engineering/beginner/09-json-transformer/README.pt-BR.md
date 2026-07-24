# Transformador de JSON

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Dois sistemas raramente concordam sobre o formato de seu JSON. Um aninha um endereço fundo dentro de um objeto de cliente; o outro quer uma linha plana com `customer_city`. Construa uma ferramenta que remodela JSON de uma estrutura de origem para uma estrutura de destino usando um mapeamento declarativo — achatando objetos aninhados, renomeando chaves, escolhendo campos dentro de arrays e convertendo tipos. O objetivo é um pequeno *motor* de transformação orientado por configuração em vez de código pontual, para que a mesma ferramenta possa adaptar o documento A no documento B sem você reescrever lógica toda vez que o esquema muda.

## Pré-requisitos

- Entendimento sólido de JSON (objetos, arrays, aninhamento, null)
- Conforto para navegar em estruturas de dados aninhadas no código
- Familiaridade com recursão ou travessia iterativa de árvores
- Tratamento de erros em nível iniciante para campos ausentes ou inesperados

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Percorrer JSON arbitrariamente aninhado com segurança, tratando caminhos ausentes
- Expressar uma transformação como um mapeamento caminho-de-origem → caminho-de-destino
- Achatar objetos aninhados em chaves com ponto ou sublinhado e o inverso (aninhar)
- Tratar arrays: escolher um elemento, mapear sobre todos ou agregar (contar, juntar)
- Converter valores para um tipo de destino e usar padrão ou rejeitar em caso de incompatibilidade
- Validar a saída contra um formato esperado antes de emiti-la

## Requisitos Funcionais

1. A ferramenta deve ler entrada JSON (um único documento ou registros delimitados por linha) e uma especificação de mapeamento.
2. Deve resolver caminhos de origem aninhados (ex.: `address.city`) e colocar valores em caminhos de destino, com caminhos ausentes tratados explicitamente.
3. Deve suportar achatamento (aninhado → plano) e aninhamento (plano → aninhado) orientados pelo mapeamento.
4. Deve tratar arrays com uma regra declarada: índice, mapear-sobre ou agregar.
5. Valores devem ser convertidos para os tipos de destino declarados; uma conversão falha deve usar padrão ou rotear o registro para rejeições conforme a config.
6. A saída transformada deve ser validada contra um formato de destino e escrita; um resumo reporta contagens de transformados e rejeitados.

## Marcos Sugeridos

1. **Marco 1 — Renomear e escolher:** Mapeie campos de nível superior e copie chaves selecionadas para o destino.
2. **Marco 2 — Aninhados e arrays:** Resolva caminhos aninhados, achate/aninhe e aplique uma regra de array.
3. **Marco 3 — Tipos e validação:** Adicione conversão de tipos, padrões/rejeições, validação de saída e um resumo.

## Esboço de Dados e Interface

```text
documento de origem
  { "id": 7,
    "customer": { "name": "Ana", "address": { "city": "Recife" } },
    "items": [ {"sku":"A","qty":2}, {"sku":"B","qty":1} ] }

especificação de mapeamento
  id                       -> order_id        (cast: int)
  customer.name            -> customer_name
  customer.address.city    -> customer_city   (default: "unknown")
  items[*].qty             -> total_qty        (aggregate: sum)
  items[0].sku             -> first_sku

documento de destino (plano)
  { "order_id": 7, "customer_name": "Ana",
    "customer_city": "Recife", "total_qty": 3, "first_sku": "A" }

em caminho ausente -> usa default OU rejeita (por config de campo)
resumo: transformados=980 rejeitados=20
```

## Desafios Extras

- Suportar uma pequena sintaxe de expressão para campos derivados (concatenação, aritmética).
- Adicionar mapeamento reverso para que uma transformação possa ser invertida (destino → origem).
- Validar contra um JSON Schema em vez de um formato ad-hoc.
- Transmitir JSON delimitado por linha para que arquivos enormes se transformem sem armazenamento em memória.

## Definição de Pronto

- [ ] Caminhos de origem aninhados resolvem corretamente, com caminhos ausentes tratados conforme a config.
- [ ] Achatar e aninhar funcionam, orientados apenas pela especificação de mapeamento.
- [ ] Regras de array (índice, mapear, agregar) produzem valores de destino corretos.
- [ ] Conversões de tipo têm sucesso ou disparam o comportamento configurado de padrão/rejeição.
- [ ] A saída é validada contra o formato de destino e um resumo é reportado.

## Armadilhas Comuns

- Travar em uma chave aninhada ausente em vez de aplicar um padrão ou rejeitar de forma limpa.
- Codificar a transformação de forma fixa, de modo que uma mudança de esquema significa editar código, não config.
- Perder dados de array pegando `[0]` quando você queria agregar sobre todos os elementos.
- Emitir saída inválida por pular a validação do formato transformado.

## Recursos

- [Especificação JSON (ECMA-404)](https://www.json.org/json-en.html) — o modelo de dados preciso que você está percorrendo.
- [JMESPath](https://jmespath.org/) — uma linguagem de consulta para extrair e remodelar JSON, ótima inspiração para sintaxe de caminhos.
- [Manual do jq](https://jqlang.github.io/jq/manual/) — a ferramenta canônica de transformação de JSON para comparar.
- [Validação com JSON Schema](https://json-schema.org/understanding-json-schema/) — como validar sua saída de destino com rigor.
