# API de Upload de Arquivos

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 5–8 horas

## Visão Geral

Construa uma API que aceita uploads de arquivos via `multipart/form-data`, armazena-os com segurança no disco local e permite que clientes os listem, baixem e removam. Pense no enviador de avatar por trás de uma página de perfil ou na caixa de anexos de um formulário de suporte. Os desafios interessantes são todos sobre segurança: validar tipo e tamanho, gerar nomes de arquivo que não possam escapar do seu diretório de armazenamento e nunca confiar no nome enviado pelo cliente.

## Pré-requisitos

- Conforto em construir endpoints HTTP que retornam JSON ([API de Notas com Persistência em Arquivo](../04-notes-api-file-persistence/) cobre E/S de arquivos)
- Entender o que é `multipart/form-data` e como difere de um corpo JSON
- Consciência de tipos MIME e extensões de arquivo
- Familiaridade com caminhos e permissões de sistema de arquivos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Analisar um upload `multipart/form-data` e acessar o stream do arquivo e seus metadados
- Impor limites de tipo e tamanho de arquivo antes de gravar qualquer coisa no disco
- Gerar nomes de armazenamento seguros e únicos e prevenir ataques de path traversal
- Armazenar e servir metadados do arquivo separadamente dos bytes
- Implementar download com os cabeçalhos `Content-Type` e `Content-Disposition` corretos

## Requisitos Funcionais

1. O sistema deve aceitar um arquivo via `multipart/form-data` e armazená-lo no disco local.
2. O sistema deve rejeitar arquivos acima de um limite de tamanho configurado com 413.
3. O sistema deve rejeitar tipos de arquivo não permitidos com 415, checando mais do que apenas a extensão.
4. Arquivos armazenados devem receber um nome único e sanitizado que não possa sobrescrever outros nem escapar do diretório de upload.
5. O sistema deve retornar metadados (id, nome original, tamanho, tipo, hora do upload) em um upload bem-sucedido.
6. O sistema deve listar os arquivos enviados e permitir baixar um pelo seu id.
7. O sistema deve permitir remover um arquivo pelo id e retornar 404 para um id desconhecido.

## Marcos Sugeridos

1. **Marco 1 — Receber e armazenar:** Aceite um upload, grave-o no disco sob um nome gerado e retorne os metadados.
2. **Marco 2 — Validar:** Imponha limites de tamanho e tipo, rejeitando uploads ruins antes que toquem o disco.
3. **Marco 3 — Servir e gerenciar:** Adicione endpoints de listar, baixar (com cabeçalhos corretos) e remover.

## Esboço de Dados e Interface

```text
POST   /files    (multipart/form-data, campo "file")
                 -> 201 { id, originalName, storedName, size, mimeType, uploadedAt }
                 -> 413 muito grande | 415 tipo não suportado
GET    /files             -> 200 [ metadata, ... ]
GET    /files/{id}        -> 200 <bytes> + Content-Disposition | 404
DELETE /files/{id}        -> 204 | 404

storedName = <uuid>.<ext-segura>   (nunca o nome cru do cliente)
```

## Desafios Extras

- Verifique o tipo real do arquivo inspecionando o cabeçalho de números mágicos, não só a extensão.
- Exija autenticação para que apenas usuários logados possam enviar ou remover.
- Gere miniaturas de imagem no upload para tipos de imagem.
- Adicione uma cota total de armazenamento e rejeite uploads que a excederiam.

## Definição de Pronto

- [ ] Um arquivo válido é enviado, armazenado sob um nome gerado e retorna metadados corretos.
- [ ] Arquivos grandes demais retornam 413 e tipos não permitidos retornam 415, antes de qualquer escrita em disco.
- [ ] Um nome forjado como `../../etc/passwd` não consegue escrever fora do diretório de upload (teste isso).
- [ ] O download retorna os bytes originais com um nome de arquivo e tipo de conteúdo sensatos.
- [ ] Remover um arquivo apaga tanto os bytes quanto seus metadados; ids desconhecidos retornam 404.

## Armadilhas Comuns

- Confiar no nome de arquivo fornecido pelo cliente e gravá-lo diretamente — isso habilita path traversal e sobrescritas.
- Checar só a extensão para o tipo, o que é trivialmente falsificável; inspecione também o conteúdo.
- Bufferizar o arquivo inteiro em memória, o que falha em uploads grandes — transmita para o disco.
- Ler o tamanho só depois de receber o arquivo por completo, de modo que o limite de tamanho nunca protege você.

## Recursos

- [MDN: Enviando dados de formulário (multipart)](https://developer.mozilla.org/pt-BR/docs/Learn/Forms/Sending_and_retrieving_form_data) — como uploads são codificados no fio.
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) — o checklist de segurança para exatamente este projeto.
- [MDN: Content-Disposition](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Content-Disposition) — controlando nomes de arquivo no download.
- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal) — o ataque que seu tratamento de nomes deve derrotar.
