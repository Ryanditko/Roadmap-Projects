# Projete um Sistema de Armazenamento de Arquivos

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um serviço que permita a usuários enviar, armazenar e baixar arquivos — um Dropbox ou S3 simplificado. A tensão interessante é que arquivos são blobs grandes e opacos, enquanto tudo o que você precisa para buscar e listar é um pequeno conjunto de metadados. Um bom design separa os dois: blobs vão para object storage, metadados vão para um banco de dados. Você vai raciocinar sobre durabilidade, como uploads grandes são tratados e como um download se mantém rápido. Entregue um documento de design que explique a separação, a API e como arquivos sobrevivem a falhas de hardware.

## Pré-requisitos

- Entendimento da diferença entre os bytes de um arquivo e seus metadados
- Consciência do que object storage (como o S3) oferece em relação a um banco de dados
- Familiaridade com upload/download HTTP e content types
- Conforto para raciocinar sobre redundância e cópias de dados

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Separar armazenamento de blobs do de metadados e justificar por quê
- Raciocinar sobre durabilidade por meio de replicação
- Projetar um fluxo de upload para arquivos grandes (chunking, URLs pré-assinadas)
- Estimar crescimento de armazenamento e banda
- Enunciar um trade-off entre simplicidade e durabilidade

## Requisitos e Restrições

1. Enviar um arquivo e receber de volta um identificador ou URL estável.
2. Baixar um arquivo pelo seu identificador.
3. Listar os arquivos de um usuário com seus metadados (nome, tamanho, hora de envio).
4. Arquivos devem sobreviver à perda de um único nó de armazenamento (durabilidade).
5. Suportar arquivos até um limite de tamanho definido; tratar os grandes com elegância.
6. Estime a escala: 1M usuários, média de 20 arquivos cada a 2 MB — cerca de 40 TB no total.

## Abordagem Sugerida

1. Desenhe a fronteira: blobs para object storage, metadados para um banco de dados.
2. Faça a conta: 1M × 20 × 2 MB = 40 TB; planeje o fator de replicação (ex., ×3 = 120 TB bruto).
3. Projete o caminho de upload — para arquivos grandes, descreva upload em chunks ou uma URL pré-assinada para que os bytes pulem seu servidor de aplicação.
4. Projete o caminho de download e considere um CDN para arquivos acessados com frequência.
5. Descreva como a replicação entrega durabilidade e o que acontece quando um nó falha.

## Esboço de Arquitetura

```text
Cliente ── pede upload ────> [ App ] -- URL pré-assinada --> Cliente
Cliente ── PUT bytes ──────> [ Object Store (replicado ×3) ]
Cliente ── GET /files/{id} > [ App ] -> metadados + [ CDN / Object Store ]

API principal
  POST /files            { name, size }   -> 201 { fileId, uploadUrl }
  PUT  <uploadUrl>       (bytes crus)      -> 200
  GET  /files/{id}                         -> 302 para blob | stream
  GET  /files            ?owner=me         -> [ { fileId, name, size, createdAt } ]

Modelo de dados
  files: file_id (PK) | owner_id | name | size | content_type | blob_key | created_at
  blob:  armazenado no object store por blob_key, replicado entre nós
```

## Tópicos de Aprofundamento

- **Durabilidade:** fator de replicação vs. erasure coding, e o que "onze noves" realmente significa.
- **Uploads grandes:** chunking, uploads retomáveis e por que os bytes devem contornar o servidor de aplicação.
- **Deduplicação:** chaves por content-hash para que arquivos idênticos compartilhem um blob (desafio extra).

## Entregáveis

- Um diagrama de arquitetura mostrando a separação blob/metadados e os caminhos de upload/download.
- O contrato da API principal para upload, download e listagem.
- Um modelo de dados separando metadados de arquivo do armazenamento de blobs.
- Um trade-off descrito: ex., armazenar blobs no banco (simples, um sistema) vs. object storage dedicado (escala, durável, mas dois sistemas a coordenar).

## Armadilhas Comuns

- Fazer streaming de cada byte pelo servidor de aplicação, tornando-o o gargalo.
- Armazenar blobs grandes em um banco relacional e esbarrar em limites de tamanho e desempenho.
- Tratar uma única cópia como segura — durabilidade exige redundância.
- Esquecer que consultas de metadados (listar, buscar) precisam do próprio store indexado.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de object storage e durabilidade.
- [AWS S3: como funciona](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) — o serviço de object storage de referência.
- [AWS: upload de objetos usando multipart upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html) — como uploads grandes são divididos em chunks.
- [Wikipedia: Erasure code](https://en.wikipedia.org/wiki/Erasure_code) — uma alternativa à replicação para durabilidade.
