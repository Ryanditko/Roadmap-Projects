# Automação de Backup

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um script que faz backup do que importa — um dump de banco de dados, um diretório de arquivos, alguma config — em um cronograma e, tão importante quanto, prova que o backup é restaurável. Um backup que você nunca restaurou é uma esperança, não uma rede de segurança. Você vai comprimir e datar cada backup, aplicar uma política de retenção para que cópias antigas sejam podadas em vez de encher o disco, e rodar uma restauração para um local descartável para verificar que o arquivo está íntegro. No caminho você conhece a regra 3-2-1 e a verdade incômoda de que a parte difícil dos backups é sempre a restauração.

## Pré-requisitos

- Algo que valha a pena fazer backup (um banco de dados, um diretório de dados ou arquivos de config)
- Uma linguagem de script ou shell com acesso à origem e a um destino
- Entendimento de compressão e arquivos compactados (tar, zip)
- Um destino com espaço suficiente (disco local, drive externo ou object storage)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Automatizar um backup agendado e datado de uma origem definida
- Comprimir arquivos e nomeá-los para que ordenem e sejam podados de forma limpa
- Aplicar uma política de retenção que mantém backups recentes e remove antigos
- Verificar um backup restaurando-o para um local temporário
- Alertar quando um backup ou verificação falha

## Requisitos Funcionais

1. O script deve fazer backup de uma origem configurada (arquivos e/ou um dump de banco) sob demanda e em um cronograma.
2. Cada backup deve ser comprimido e nomeado com um timestamp UTC ordenável.
3. Uma política de retenção deve manter os últimos N (ou N dias de) backups e apagar os mais antigos.
4. O script deve verificar cada backup, no mínimo testando a integridade do arquivo.
5. Um caminho de restauração deve existir e ser exercitado em um local temporário, não sobre dados em produção.
6. O script deve sair com código diferente de zero e alertar em qualquer falha de backup ou verificação.
7. Origem, destino, retenção e cronograma devem ser configuráveis.

## Marcos Sugeridos

1. **Marco 1 — Snapshot:** Arquive a origem em um arquivo comprimido e datado sob demanda.
2. **Marco 2 — Retenção e cronograma:** Pode backups antigos por política e rode o backup em um cronograma.
3. **Marco 3 — Verificar e restaurar:** Teste a integridade do arquivo e realize uma restauração em um local descartável.

## Esboço de Dados e Interface

```text
Config (estrutura, não o arquivo completo)
  source:
    files: [/etc/app, /srv/app/data]
    database: { type: postgres, name: appdb }   # via dump, não arquivos crus
  destination: /backups            (ou s3://bucket/prefix)
  retention: { keep_days: 7, keep_min: 3 }
  schedule: "0 2 * * *"            # expressão cron, agendador externo

Nomeação do artefato
  app-backup-YYYYMMDDTHHMMSSZ.tar.gz

Fluxo (estrutura, não o script completo)
  dump do db -> arquiva arquivos + dump -> comprime -> escreve no destino
  verifica: testa integridade do arquivo (e restauração opcional em /tmp/verify)
  poda: lista backups -> mantém por retenção -> apaga o resto
  em qualquer falha: loga + alerta + sai != 0
```

## Desafios Extras

- Criptografe arquivos em repouso e gerencie a chave fora do próprio backup.
- Adicione backups incrementais ou diferenciais para reduzir o tamanho diário.
- Envie uma cópia off-site (object storage) para satisfazer a regra 3-2-1.
- Emita uma métrica ou relatório: horário do último backup bem-sucedido e tamanho total.

## Definição de Pronto

- [ ] Um backup roda sem supervisão no cronograma e produz um arquivo datado e comprimido.
- [ ] A retenção poda backups antigos sem nunca apagar o mais recente válido.
- [ ] A integridade do arquivo é verificada após cada execução.
- [ ] Uma restauração em um local temporário reproduz os dados originais.
- [ ] Um backup ou verificação com falha sai com código diferente de zero e dispara um alerta.

## Armadilhas Comuns

- Fazer backup de arquivos crus do banco enquanto ele está rodando, produzindo um snapshot inconsistente — use dump.
- Nunca testar uma restauração, de modo que a primeira restauração real também é a primeira vez que você descobre que ela não funciona.
- Um bug de retenção que apaga tudo, inclusive o backup de que você precisa, quando a lista está vazia ou mal interpretada.
- Guardar backups no mesmo disco da origem, de modo que uma falha de disco leva ambos.

## Recursos

- [US-CERT: Opções de Backup de Dados (regra 3-2-1)](https://www.cisa.gov/sites/default/files/publications/data_backup_options.pdf) — a estratégia fundamental.
- [Manual do GNU tar](https://www.gnu.org/software/tar/manual/tar.html) — arquivamento e backups incrementais.
- [PostgreSQL: Backup e Restauração](https://www.postgresql.org/docs/current/backup.html) — dumps de banco consistentes.
- [Documentação do restic](https://restic.readthedocs.io/) — uma ferramenta moderna que modela retenção e verificação bem.
