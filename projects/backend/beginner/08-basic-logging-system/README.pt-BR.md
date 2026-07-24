# Sistema de Logging Básico

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 4–7 horas

## Visão Geral

Construa uma pequena biblioteca de logging que registra eventos da aplicação em arquivos com níveis de severidade, timestamps e rotação de arquivos — aquilo em que todo serviço de produção se apoia, mas que poucos iniciantes constroem do zero. Ao criar seu próprio logger você aprende o que frameworks como Winston, Bunyan ou o `logging` do Python de fato fazem: filtrar por nível, formatar de forma consistente, escrever com segurança e impedir que um arquivo de log cresça para sempre. Depois você o conecta a um app de exemplo para vê-lo funcionar no contexto.

## Pré-requisitos

- Conforto com a E/S de arquivos da sua linguagem
- Entendimento dos níveis de severidade padrão (DEBUG, INFO, WARN, ERROR)
- Familiaridade com timestamps e a formatação ISO 8601
- Um pequeno app existente (qualquer projeto anterior) para instrumentar

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar níveis de severidade de log e filtrar a saída por um limiar configurável
- Produzir linhas de log com formatação consistente contendo timestamp, nível e mensagem
- Escrever logs em um arquivo com segurança, anexando sem perder o conteúdo existente
- Rotacionar arquivos de log por tamanho ou data para que nunca cresçam sem limite
- Comparar formatos de log legíveis por humanos e estruturados (JSON) e quando cada um se encaixa

## Requisitos Funcionais

1. O sistema deve suportar ao menos quatro níveis de severidade com uma ordem definida.
2. O sistema deve suprimir mensagens abaixo de um nível mínimo configurável.
3. Toda linha de log deve incluir um timestamp ISO 8601, o nível e a mensagem.
4. Os logs devem ser anexados a um arquivo, preservando entradas anteriores entre reinícios.
5. O sistema deve rotacionar o arquivo de log quando ele exceder um limiar de tamanho (ou em uma virada de data).
6. Arquivos rotacionados devem ser retidos até uma quantidade configurável, com o mais antigo removido.
7. O nível mínimo e o destino de saída devem ser configuráveis sem alterações de código.

## Marcos Sugeridos

1. **Marco 1 — Logging com níveis:** Emita linhas formatadas e com timestamp e respeite um filtro de nível mínimo.
2. **Marco 2 — Saída em arquivo:** Anexe logs a um arquivo que sobrevive a reinícios e então instrumente um app de exemplo.
3. **Marco 3 — Rotação:** Rotacione por tamanho, mantenha N arquivos antigos, apague o resto; opcionalmente adicione um formato JSON.

## Esboço de Dados e Interface

```text
Ordem dos níveis: DEBUG < INFO < WARN < ERROR

Formato de linha em texto:
  2026-07-24T14:03:22Z  INFO   user login succeeded  {userId=42}

Formato de linha em JSON:
  { "ts": "...", "level": "INFO", "msg": "...", "ctx": { "userId": 42 } }

Rotação:
  app.log cresce além de MAX_BYTES
    -> renomear app.log -> app.log.1  (deslocar .1 -> .2, ...)
    -> manter no máximo KEEP arquivos, apagar os mais antigos
```

## Desafios Extras

- Adicione um modo de saída JSON alternável por configuração, ao lado do formato de texto.
- Suporte múltiplos destinos simultâneos (console + arquivo) com níveis independentes.
- Adicione campos contextuais (id de requisição, id de usuário) carregados por uma chamada de log.
- Adicione rotação baseada em tempo (diária) além da baseada em tamanho.

## Definição de Pronto

- [ ] Mensagens abaixo do nível configurado não são escritas.
- [ ] Toda linha tem um timestamp ISO 8601 válido e seu nível.
- [ ] Os logs persistem e são anexados corretamente entre reinícios.
- [ ] O arquivo rotaciona no limiar de tamanho e arquivos antigos são podados até a quantidade configurada.
- [ ] Nível e destino são definidos via configuração, não fixados no código.

## Armadilhas Comuns

- Abrir o arquivo em modo de truncamento em vez de anexação, apagando o histórico a cada início.
- Formatar timestamps em hora local sem um offset, tornando os logs ambíguos entre máquinas.
- Rotacionar renomeando enquanto uma escrita está em andamento, perdendo ou intercalando linhas — coordene a troca.
- Comparar níveis como strings em vez de valores ordenados, fazendo o filtro se comportar de forma imprevisível.

## Recursos

- [Python logging HOWTO](https://docs.python.org/3/howto/logging.html) — o modelo canônico de níveis e handlers.
- [The Twelve-Factor App: Logs](https://12factor.net/pt_br/logs) — tratando logs como fluxos de eventos.
- [RFC 5424: Níveis de severidade do Syslog](https://datatracker.ietf.org/doc/html/rfc5424#section-6.2.1) — a origem dos níveis padrão.
- [Wikipedia: Rotação de logs](https://en.wikipedia.org/wiki/Log_rotation) — as estratégias de rotação que você está implementando.
