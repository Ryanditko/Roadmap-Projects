# Pipeline de Limpeza de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Dados do mundo real são bagunçados: o mesmo cliente escrito de três formas, datas em cinco formatos, números de telefone com e sem código de país, células em branco que significam coisas diferentes. Construa um pipeline que pega um conjunto de dados sujo e produz um limpo e normalizado — deduplicando registros, preenchendo ou sinalizando valores ausentes e padronizando formatos. A disciplina aqui é tornar cada decisão de limpeza *explícita e reversível*: você mantém uma trilha de auditoria do que mudou e por quê, para que um analista downstream possa confiar na saída e rastrear qualquer valor de volta à sua forma bruta.

## Pré-requisitos

- Capacidade de ler e escrever um conjunto de dados CSV ou JSON
- Conforto com manipulação de strings e tipos de dados básicos
- Entender o que "duplicata" e "ausente" significam para seus dados
- Familiaridade com dicionários/mapas para agrupar registros

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Detectar registros duplicados por uma chave exata e por uma chave normalizada/fuzzy
- Escolher e aplicar uma estratégia de valores ausentes (descartar, padrão ou sinalizar) por coluna
- Normalizar formatos: espaços, caixa, datas e unidades numéricas para uma forma canônica
- Manter uma trilha de auditoria registrando cada transformação aplicada a cada registro
- Produzir métricas de qualidade antes/depois para que a limpeza seja mensurável
- Separar registros "limpos" dos "não corrigíveis" em vez de forçar toda linha a passar

## Requisitos Funcionais

1. O pipeline deve remover linhas duplicadas exatas e detectar quase-duplicatas por uma chave normalizada.
2. Deve aplicar uma estratégia configurável de valores ausentes por coluna, nunca adivinhando silenciosamente.
3. Deve normalizar ao menos: espaços em branco, caixa das letras, formatos de data e um campo numérico/de unidade.
4. Toda mudança deve ser registrada em uma trilha de auditoria ligando o valor limpo ao seu original.
5. Registros que não podem ser limpos para atender às regras devem ser roteados para uma saída de rejeições, não forçados.
6. O pipeline deve reportar métricas de qualidade antes e depois: completude, taxa de duplicatas e linhas rejeitadas.

## Marcos Sugeridos

1. **Marco 1 — Normalizar:** Aplique normalização de espaços, caixa e formato a cada campo.
2. **Marco 2 — Deduplicar e ausentes:** Remova duplicatas e aplique estratégias de valores ausentes por coluna.
3. **Marco 3 — Auditoria e métricas:** Adicione a trilha de auditoria, o roteamento de rejeições e as métricas de qualidade antes/depois.

## Esboço de Dados e Interface

```text
linha de origem (suja)
  { "name": "  ANA souza ", "email": "ANA@example.com",
    "phone": "(81) 9999-1111", "signup": "01/05/2024" }

etapas de limpeza
  name   -> trim + title-case        -> "Ana Souza"
  email  -> trim + minúsculas        -> "ana@example.com"
  phone  -> remove não-dígitos + E.164 -> "+558199991111"
  signup -> parse dd/mm/yyyy -> ISO  -> "2024-05-01"

chave de dedup: lower(email)  -> colapsa duplicatas, mantém a mais nova

trilha de auditoria (por registro)
  { id: 7, changes: ["name:trim+case", "signup:reformatado"] }

rejeições -> unfixable.jsonl  (ex.: phone sem dígitos)

métricas
  antes:  linhas=1000 completude=82% taxa_duplicatas=6%
  depois: linhas=940  completude=99% taxa_duplicatas=0% rejeitadas=12
```

## Desafios Extras

- Adicionar correspondência fuzzy (distância de edição) para pegar nomes quase-duplicados, não só chaves exatas.
- Tornar as regras de limpeza orientadas por configuração para que o mesmo motor trate diferentes conjuntos de dados.
- Suportar modos de limpeza "suave" vs "forte" (só sinalizar vs modificar no lugar).
- Emitir um relatório de diff mostrando valores de amostra antes/depois para conferência.

## Definição de Pronto

- [ ] Registros duplicados exatos e quase-duplicados são colapsados pela chave escolhida.
- [ ] A estratégia de valores ausentes de cada coluna é aplicada explicitamente, não por acidente.
- [ ] Espaços, caixa, datas e o campo numérico são normalizados de forma consistente.
- [ ] Todo valor alterado é rastreável até seu original pela trilha de auditoria.
- [ ] Métricas de qualidade antes/depois e uma contagem de rejeições são reportadas.

## Armadilhas Comuns

- Preencher valores ausentes com um padrão que polui a análise (ex.: `0` onde NULL significa "desconhecido").
- Deduplicar pela chave bruta, deixando "Ana" e "ANA " sobreviverem como dois registros.
- Aplicar uma transformação irreversível sem registro do valor original.
- Forçar linhas não corrigíveis a passar em vez de colocá-las em quarentena, corrompendo o conjunto limpo.

## Recursos

- [pandas: Trabalhando com dados ausentes](https://pandas.pydata.org/docs/user_guide/missing_data.html) — estratégias para nulos bem feitas.
- [Formato de número de telefone E.164](https://en.wikipedia.org/wiki/E.164) — o padrão internacional canônico de telefone.
- [Formas de normalização Unicode](https://unicode.org/reports/tr15/) — por que texto precisa ser canonizado antes da comparação.
- [Tidy Data (Hadley Wickham)](https://vita.had.co.nz/papers/tidy-data.pdf) — o artigo que define o que dados tabulares "limpos" significam.
