# Editor Colaborativo em Tempo Real (como o Google Docs)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um editor de documentos onde várias pessoas digitam no mesmo documento ao mesmo tempo e cada tecla aparece na tela de todos em instantes — a experiência que o Google Docs, o Notion e o Figma popularizaram. A demo enganosamente simples esconde o problema mais difícil do software colaborativo: dois usuários editam o mesmo ponto offline, reconectam, e o sistema deve fundir ambas as intenções em um documento consistente sem um lock central. Você vai se apoiar em um algoritmo de convergência comprovado (CRDT ou transformação operacional) em vez de inventar o seu, e gastar sua energia nas preocupações de frontend que fazem tudo parecer vivo: cursores remotos, presença e um editor que nunca trava o digitador local esperando a rede.

## Pré-requisitos

- Conforto em construir apps interativos com estado em tempo real (uma [Aplicação de Chat](../../intermediate/04-chat-ui/) é um bom degrau)
- Conhecimento prático de WebSockets e fluxo de dados orientado a eventos
- Entendimento das APIs Selection e Range do navegador, ou de um framework de rich-text
- Consciência de que o ingênuo "last write wins" perde dados — a motivação deste projeto

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Explicar por que edições concorrentes precisam de CRDTs ou OT em vez de um esquema simples de lock
- Integrar uma biblioteca de convergência (Yjs ou Automerge) com uma superfície de edição rich-text
- Renderizar cursores e seleções remotas mapeados para posições no documento vivo
- Manter as edições locais instantâneas (otimistas) enquanto a sincronização em segundo plano reconcilia o estado
- Preservar edições feitas offline e fundi-las de forma limpa na reconexão

## Requisitos Funcionais

1. Dois ou mais clientes editando o mesmo documento devem convergir para conteúdo idêntico após todas as mudanças se propagarem.
2. Uma edição local deve aparecer instantaneamente, sem esperar por uma ida e volta ao servidor.
3. O cursor e a seleção de texto de cada usuário conectado devem ser visíveis aos demais, rotulados e coloridos por usuário.
4. Uma lista de presença deve mostrar quem está atualmente no documento e atualizar ao entrar/sair.
5. Edições feitas offline devem ser retidas e fundidas automaticamente assim que a conexão retornar.
6. Edições concorrentes na mesma região devem fundir de forma determinística, nunca descartando silenciosamente a entrada de um usuário.
7. Desfazer/refazer deve operar sobre as próprias mudanças do usuário local sem reverter as edições dos outros.

## Marcos Sugeridos

1. **Marco 1 — Editor de usuário único + transporte:** Construa a superfície de edição e um canal WebSocket que ecoa as mudanças.
2. **Marco 2 — Convergência:** Adote uma biblioteca CRDT/OT para que dois clientes fundam edições concorrentes corretamente.
3. **Marco 3 — Presença e cursores:** Transmita e renderize cursores remotos, seleções e uma lista de presença ao vivo.
4. **Marco 4 — Offline e histórico:** Enfileire edições offline, funda na reconexão e adicione desfazer/refazer por usuário.

## Esboço de Dados e Interface

```text
   Cliente A                    Servidor de sync             Cliente B
 ┌──────────┐   ops locais    ┌───────────────┐   ops      ┌──────────┐
 │ editor   │ ───────────────▶│ relay + estado│───────────▶│ editor   │
 │ doc CRDT │◀─────────────── │ do doc (opc.) │◀───────────│ doc CRDT │
 └────┬─────┘   ops remotas   └───────────────┘            └────┬─────┘
      │ aplicação otimista (instantânea, local-first)           │
      └── presença: { userId, nome, cor, cursor, selection } ───┘

Op (conceitual):  { type: insert|delete, pos, value?, origin, lamport }
Awareness:        efêmero, não persistido — cursores, presença, digitação
Convergência:     CRDT (Yjs / Automerge) ou OT — escolha e justifique

Metas não funcionais:
  tecla local -> na tela        < 16 ms (sem espera de rede)
  edição -> visível no par      < 250 ms em um link saudável
  edições offline               nunca perdidas na reconexão
```

## Desafios Extras

- Adicione um histórico de versões com snapshots nomeados e uma visão de diff entre revisões.
- Suporte comentários e sugestões inline ancorados a um intervalo de texto que sobrevivem às edições.
- Adicione permissões em nível de documento (visualizar / comentar / editar) aplicadas no servidor.
- Mostre um estado de "reconectando" com um contador da fila de edições, depois uma animação limpa de recuperação.

## Definição de Pronto

- [ ] Dois navegadores editando simultaneamente terminam com documentos byte a byte idênticos.
- [ ] A digitação parece instantânea mesmo com latência de rede artificial adicionada no dev tools.
- [ ] Cursores remotos acompanham a posição de caractere correta conforme texto é inserido acima deles.
- [ ] Desconectar um cliente, editar em ambos e reconectar funde sem perda de dados.
- [ ] Desfazer reverte apenas a última ação do usuário local, deixando as edições remotas intactas.

## Armadilhas Comuns

- Reinventar a transformação operacional do zero — é notoriamente sutil; use uma biblioteca testada.
- Armazenar posições de cursor como offsets absolutos, apontando para o lugar errado após inserções remotas.
- Persistir dados efêmeros de awareness (cursores, presença) no documento e inchá-lo.
- Bloquear a UI esperando o reconhecimento do servidor, destruindo a sensação de digitação instantânea.
- Assumir entrega ordenada e confiável — redes reordenam e descartam; a fusão não pode depender da ordem de chegada.

## Recursos

- [Documentação do Yjs](https://docs.yjs.dev/) — um framework CRDT maduro com bindings de editor e um protocolo de awareness.
- [Automerge](https://automerge.org/) — uma biblioteca CRDT alternativa com um forte modelo de estrutura de dados.
- [Martin Kleppmann: CRDTs — the hard parts](https://www.youtube.com/watch?v=x7drE24geUw) — uma palestra rigorosa sobre garantias de convergência.
- [MDN: Selection API](https://developer.mozilla.org/pt-BR/docs/Web/API/Selection) — a primitiva do navegador por trás do tratamento de cursor e intervalo.
