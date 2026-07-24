# Analisador de Logs

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Logs de servidor chegam como paredes de texto, mas escondido dentro de cada linha há um registro estruturado: um timestamp, um nível, uma origem, uma mensagem. Construa uma ferramenta que lê um arquivo de log, extrai esses campos em registros limpos e permite filtrá-los e resumi-los — quantos erros na última hora, qual endpoint está mais lento, quais são as mensagens mais frequentes. Esta é a habilidade fundamental por trás de todo pipeline de observabilidade: transformar texto livre em dados consultáveis. O coração do projeto é um analisador robusto que lida com as linhas que *quase* correspondem ao formato sem quebrar.

## Pré-requisitos

- Conforto para ler arquivos linha a linha
- Expressões regulares em nível iniciante ou conhecimento de um formato de log estruturado (JSON lines)
- Entendimento básico de timestamps e intervalos de tempo
- Familiaridade com dicionários/mapas para agrupar e contar

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Definir um formato de linha de log e analisá-lo em campos nomeados
- Lidar tanto com um formato de texto fixo (via regex) quanto com JSON lines estruturado
- Tratar com elegância as linhas que não correspondem — conte-as, não trave
- Filtrar registros por nível, janela de tempo ou valor de campo
- Agregar: contagens por nível, top-N mensagens, requisições por minuto
- Ler um arquivo grande em streaming sem carregá-lo inteiro na memória

## Requisitos Funcionais

1. A ferramenta deve analisar cada linha em um registro com ao menos timestamp, nível e mensagem.
2. Deve suportar um formato de linha configurável (padrão regex ou JSON) em vez de codificar um fixo.
3. Linhas que falharem na análise devem ser contadas e opcionalmente gravadas em um arquivo de "não analisadas", nunca descartadas silenciosamente.
4. A ferramenta deve filtrar registros por nível e por um intervalo de tempo informado na linha de comando.
5. Deve produzir agregações: contagem por nível e as top-N mensagens mais frequentes.
6. Deve processar arquivos linha a linha para que logs arbitrariamente grandes sejam suportados.

## Marcos Sugeridos

1. **Marco 1 — Analisar:** Transforme cada linha correspondente em um registro estruturado e imprima-o.
2. **Marco 2 — Filtrar:** Adicione filtragem por nível e por intervalo de tempo sobre os registros analisados.
3. **Marco 3 — Agregar e reportar:** Adicione contagens por nível, top-N mensagens e o tratamento de linhas não analisadas.

## Esboço de Dados e Interface

```text
linha de origem (estilo Apache)
  2024-05-01T10:15:03Z ERROR api order-service "db timeout" 503

padrão de análise (regex, grupos nomeados)
  (?P<ts>\S+) (?P<level>\w+) (?P<comp>\S+) (?P<svc>\S+) "(?P<msg>[^"]*)" (?P<code>\d+)

registro estruturado
  { ts, level, component, service, message, status }

linhas não analisadas -> unparsed.log (com número da linha)

fluxo de consulta
  ler -> analisar -> [ok] filtrar(level>=WARN, ts na janela) -> agregar
                  \-> [sem match] contar + destino de não analisadas

relatório
  por_nível: INFO=8210 WARN=145 ERROR=37
  top_mensagens: "db timeout" x22, "retry" x15
```

## Desafios Extras

- Detectar o formato automaticamente (JSON vs texto) farejando as primeiras linhas.
- Adicionar um modo `--follow` que acompanha o arquivo e analisa novas linhas conforme chegam.
- Calcular percentis de latência se houver um campo de duração.
- Suportar arquivos de log compactados com gzip de forma transparente.

## Definição de Pronto

- [ ] Linhas bem-formadas são analisadas em registros com todos os campos esperados.
- [ ] Linhas sem correspondência são contadas e preservadas, não descartadas silenciosamente.
- [ ] Os filtros de nível e de intervalo de tempo retornam o subconjunto correto.
- [ ] As contagens por nível e as top-N mensagens estão corretas.
- [ ] Um arquivo de centenas de MB é processado sem esgotar a memória.

## Armadilhas Comuns

- Uma regex frágil que quebra em campos entre aspas, espaços extras ou mensagens multilinha.
- Analisar timestamps sem tratar fusos horários, deixando os filtros de intervalo de tempo errados.
- Carregar o arquivo inteiro em uma lista antes de processar logs grandes.
- Pular silenciosamente linhas não analisadas e esconder uma mudança de formato que você deveria conhecer.

## Recursos

- [Módulo `re` do Python](https://docs.python.org/3/library/re.html) — grupos nomeados tornam os padrões de log legíveis.
- [regex101](https://regex101.com/) — construa e teste padrões de log de forma interativa.
- [The Twelve-Factor App: Logs](https://12factor.net/logs) — por que tratar logs como fluxos de eventos importa.
- [Elastic: processador Grok](https://www.elastic.co/guide/en/elasticsearch/reference/current/grok-processor.html) — como um sistema de produção nomeia padrões de campos de log.
