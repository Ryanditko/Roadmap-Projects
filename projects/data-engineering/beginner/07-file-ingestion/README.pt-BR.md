# Sistema de Ingestão de Arquivos

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Muitos pipelines começam com uma "pasta de entrega": outra equipe, um parceiro ou um job de exportação deixa arquivos em um diretório, e seu sistema precisa notá-los, processá-los e tirá-los do caminho. Construa um serviço que observa um diretório de entrada, pega cada arquivo novo, valida e processa, e depois o move para uma pasta de "concluídos" ou "falhos" para que nunca seja processado duas vezes. Os desafios sutis — um arquivo que ainda está sendo escrito quando você o pega, duas execuções disputando o mesmo arquivo, um arquivo corrompido que não pode bloquear a fila — são exatamente o que torna a ingestão um problema de engenharia de verdade.

## Pré-requisitos

- Conforto para listar diretórios e mover/renomear arquivos
- Entender caminhos de arquivo e operações básicas de sistema de arquivos
- Tratamento de erros e logging em nível iniciante
- Consciência de que um arquivo pode estar parcialmente escrito (ainda não completo)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Detectar novos arquivos em um diretório observado de forma confiável
- Evitar pegar um arquivo que ainda está sendo escrito (verificação de estabilidade)
- Processar cada arquivo exatamente uma vez e movê-lo para uma pasta terminal
- Rotear arquivos com falha para uma pasta de quarentena em vez de repetir para sempre
- Tornar a captura segura contra uma segunda execução disputando o mesmo arquivo
- Registrar em log todo evento de ingestão para que o pipeline seja auditável

## Requisitos Funcionais

1. O sistema deve varrer um diretório `incoming/` e detectar arquivos que ainda não foram processados.
2. Deve confirmar que um arquivo está completo antes de processar (ex.: tamanho estável entre duas verificações, ou uma convenção de renomeação atômica).
3. Cada arquivo deve ser validado (tipo/extensão e estrutura básica) antes de ser processado.
4. Em sucesso, o arquivo deve mover para `processed/`; em falha, para `failed/` com um registro de erro.
5. Um arquivo nunca deve ser processado duas vezes, mesmo entre reinícios ou execuções sobrepostas.
6. Toda ação (detectado, processado, falho) deve ser registrada com um timestamp e o nome do arquivo.

## Marcos Sugeridos

1. **Marco 1 — Detectar e mover:** Varra `incoming/`, processe cada arquivo e mova-o para `processed/`.
2. **Marco 2 — Segurança:** Adicione a verificação de completude e um mecanismo de reivindicação para que o mesmo arquivo não seja processado em dobro.
3. **Marco 3 — Tratamento de falhas:** Valide arquivos, coloque os ruins em quarentena em `failed/` e registre cada evento.

## Esboço de Dados e Interface

```text
layout de diretórios
  incoming/   <- arquivos chegam aqui
  processing/ <- arquivos reivindicados (em andamento)
  processed/  <- sucesso
  failed/     <- quarentena + <nome>.error.txt

fluxo por arquivo
  detecta incoming/orders_2024-05-01.csv
  verificação de completude (tamanho estável? ou marcador .ready?)
  reivindica: rename atômico -> processing/  (vence a disputa)
  valida (extensão=csv, cabeçalho presente)
  processa -> [ok] move para processed/
           \-> [ruim] move para failed/ + escreve motivo

ingest.log
  2024-05-01T09:00 detectado orders_...csv
  2024-05-01T09:00 processado orders_...csv linhas=812
```

## Desafios Extras

- Adicionar um laço de observação contínua (intervalo de polling ou eventos de sistema de arquivos do SO) em vez de uma varredura única.
- Suportar múltiplos tipos de arquivo com um handler escolhido pela extensão.
- Adicionar uma política de retry com um número máximo de tentativas antes da quarentena.
- Emitir um resumo diário de arquivos ingeridos, linhas processadas e falhas.

## Definição de Pronto

- [ ] Novos arquivos em `incoming/` são detectados e processados.
- [ ] Um arquivo ainda sendo escrito não é pego até estar completo.
- [ ] Arquivos bem-sucedidos caem em `processed/`, falhas em `failed/` com um motivo.
- [ ] Nenhum arquivo é processado duas vezes, mesmo com duas execuções sobrepostas.
- [ ] Todo evento de ingestão é registrado com timestamp e nome do arquivo.

## Armadilhas Comuns

- Pegar um arquivo no meio da escrita e processar uma cópia truncada e corrompida.
- Duas execuções reivindicando o mesmo arquivo porque a reivindicação não é atômica.
- Deixar arquivos com falha em `incoming/`, de modo que são repetidos para sempre e bloqueiam a fila.
- Confiar apenas na hora de modificação para detectar arquivos "novos", o que é frágil entre relógios.

## Recursos

- [Python `pathlib`](https://docs.python.org/3/library/pathlib.html) — operações de sistema de arquivos limpas e multiplataforma.
- [Biblioteca watchdog](https://python-watchdog.readthedocs.io/en/stable/) — notificações de eventos de sistema de arquivos em nível de SO.
- [Atomicidade do `rename` POSIX](https://pubs.opengroup.org/onlinepubs/9699919799/functions/rename.html) — por que um rename atômico é a primitiva de reivindicação segura.
- [The Log: What every engineer should know](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying) — ingestão como a frente de um sistema de dados.
