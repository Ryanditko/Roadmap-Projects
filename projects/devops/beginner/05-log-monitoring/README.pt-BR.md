# Script de Monitoramento de Logs

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um script que observa os arquivos de log de uma aplicação, identifica problemas conforme acontecem e avisa alguém antes que um usuário precise. Ele acompanha o log, compara linhas com padrões que você define (erros, stack traces, requisições lentas), conta-os ao longo de uma janela e dispara um alerta quando um limiar é ultrapassado. A arte está nos detalhes: acompanhar um arquivo que rotaciona por baixo dos seus pés, evitar uma enxurrada de alertas duplicados e ajustar limiares para que problemas reais disparem enquanto o ruído comum fica quieto. Ao terminar, você terá os pequenos e confiáveis olhos-nos-logs que todo serviço precisa antes de ganhar uma stack de observabilidade completa.

## Pré-requisitos

- Uma app que escreve logs em um arquivo (ou um log de amostra ao qual você pode anexar)
- Uma linguagem de script confortável com texto (Python, Bash + ferramentas ou similar)
- Entendimento de expressões regulares para casamento de padrões
- Um lugar para enviar alertas (console, arquivo ou um webhook de chat)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Acompanhar um arquivo de log escrito ativamente, inclusive durante a rotação
- Comparar linhas com padrões configuráveis e classificar a severidade
- Contar casamentos dentro de uma janela de tempo e alertar em um limiar
- Suprimir alertas duplicados para que um incidente não vire spam
- Ajustar limiares para equilibrar sensibilidade e falsos positivos

## Requisitos Funcionais

1. O script deve acompanhar um arquivo de log em tempo real e processar novas linhas conforme chegam.
2. Ele deve comparar linhas com uma lista configurável de padrões com severidades.
3. Ele deve contar casamentos ao longo de uma janela de tempo móvel por padrão.
4. Ele deve disparar um alerta quando a contagem de um padrão exceder seu limiar na janela.
5. Ele deve deduplicar ou limitar a taxa de alertas para que uma condição alerte uma vez, não por linha.
6. Ele deve continuar acompanhando o arquivo corretamente após a rotação do log.
7. Padrões, limiares e destinos de alerta devem ser configuráveis, não hardcoded.

## Marcos Sugeridos

1. **Marco 1 — Acompanhar e casar:** Acompanhe o arquivo e imprima linhas que casam com qualquer padrão configurado.
2. **Marco 2 — Janela e limiar:** Conte casamentos ao longo de uma janela e alerte quando um limiar é ultrapassado.
3. **Marco 3 — Dedup e rotação:** Limite a taxa de alertas repetidos e trate um log rotacionado sem perder linhas.

## Esboço de Dados e Interface

```text
Config (estrutura, não o arquivo completo)
  log_file: /var/log/app/app.log
  window_seconds: 60
  patterns:
    - name: error_spike
      regex: "ERROR|FATAL"
      severity: high
      threshold: 10          # casamentos por janela
    - name: slow_request
      regex: "duration_ms=([0-9]{4,})"
      severity: medium
      threshold: 5
  alert:
    channel: webhook|console|file
    cooldown_seconds: 300    # janela de dedup por padrão

Payload do alerta
  { pattern, severity, count, window, sample_line, timestamp }
```

## Desafios Extras

- Extraia campos numéricos (ex.: latência) e alerte em uma média ou percentil, não só uma contagem.
- Resuma sob demanda as principais assinaturas de erro da última hora.
- Suporte múltiplos arquivos de log e marque alertas com sua origem.
- Adicione um modo de replay `--since` para testar regras contra logs históricos.

## Definição de Pronto

- [ ] Novas linhas de log são processadas dentro de um segundo após serem escritas.
- [ ] Ultrapassar um limiar configurado produz exatamente um alerta por janela de cooldown.
- [ ] Rotacionar o log (renomear + recriar) não interrompe o monitoramento nem duplica linhas.
- [ ] Padrões e limiares podem ser alterados via config sem editar código.
- [ ] Ruído comum de log abaixo do limiar não produz alertas.

## Armadilhas Comuns

- Reler o arquivo inteiro a cada poll em vez de rastrear a posição, desperdiçando CPU e realertando.
- Manter o handle do arquivo antigo após a rotação e monitorar silenciosamente um arquivo em que ninguém mais escreve.
- Alertar por linha casada, soterrando o sinal real sob centenas de duplicatas.
- Escrever regexes gananciosas que casam muito mais do que o pretendido e inflam contagens.

## Recursos

- [The Art of Monitoring (conceitos)](https://artofmonitoring.com/) — limiares, sinais e design de alertas.
- [Manual do logrotate](https://man7.org/linux/man-pages/man8/logrotate.8.html) — como a rotação funciona, para você sobreviver a ela.
- [MDN: Guia de expressões regulares](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Guide/Regular_expressions) — fundamentos de casamento de padrões.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — o que vale a pena alertar.
