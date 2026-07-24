# Interface de Upload de Arquivos (Progresso e Preview)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa uma interface de upload de arquivos com arrastar e soltar, previews por arquivo, barras de progresso ao vivo e a capacidade de cancelar ou repetir um upload que falhou. Qualquer um consegue plugar um `<input type="file">`; o trabalho interessante é tudo ao redor — validar tipo e tamanho antes que um byte deixe o navegador, rastrear o progresso real do upload (o que o `fetch` sozinho não consegue reportar), gerenciar uma fila de uploads concorrentes sem saturar a conexão e limpar as object URLs que você cria para os previews para que a aba não vaze memória. É um estudo focado da File API, do progresso de upload e de UI assíncrona resiliente.

## Pré-requisitos

- Conforto com estado de componente e listas em um framework (React, Vue, Svelte ou Angular)
- Familiaridade com os objetos File e Blob e com `FileReader`/`URL.createObjectURL`
- Entendimento dos eventos de progresso de upload do `XMLHttpRequest` (ou a alternativa de streaming do Fetch)
- Conhecimento básico de promessas e cancelamento (`AbortController`)
- Conhecimento dos eventos de arrastar e soltar para uma zona de drop

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Aceitar arquivos tanto por um file input quanto por uma zona de arrastar e soltar
- Validar tipo e tamanho de arquivo no cliente e rejeitar arquivos ruins com mensagens claras
- Reportar o progresso de upload real por arquivo usando eventos de progresso do `XMLHttpRequest`
- Gerenciar uma fila com limite de concorrência em vez de disparar todos os uploads de uma vez
- Cancelar um upload em andamento e repetir um que falhou
- Gerar e revogar object URLs para previews a fim de evitar vazamentos de memória

## Requisitos Funcionais

1. O usuário deve adicionar arquivos clicando num input e arrastando arquivos para uma zona de drop que reage visualmente.
2. Cada arquivo deve ser validado contra uma lista de tipos permitidos e um tamanho máximo, com um erro por arquivo na rejeição.
3. Arquivos de imagem devem mostrar uma miniatura de preview; não-imagens devem mostrar um ícone de tipo e metadados (nome, tamanho).
4. Cada arquivo em upload deve mostrar uma barra de progresso ao vivo refletindo os bytes realmente enviados.
5. Os uploads devem passar por uma fila com concorrência limitada (ex.: 3 por vez).
6. O usuário deve poder cancelar um upload em andamento e repetir um que falhou.
7. As object URLs de preview devem ser revogadas quando um arquivo é removido ou o upload é concluído.

## Marcos Sugeridos

1. **Marco 1 — Selecionar e preview:** Trate a seleção por input + zona de drop e renderize previews validados.
2. **Marco 2 — Upload e progresso:** Faça upload de cada arquivo com uma barra de progresso real via `XMLHttpRequest`.
3. **Marco 3 — Fila:** Limite a concorrência e processe a fila conforme os espaços liberam.
4. **Marco 4 — Controle:** Adicione cancelar, repetir e limpeza de object URLs.

## Esboço de Dados e Interface

```text
Layout
  ┌───────────────────────────────────────────┐
  │   ⬆  Arraste arquivos aqui ou [ procurar ] │
  ├───────────────────────────────────────────┤
  │ [img] foto.jpg    1.2 MB  ▓▓▓▓▓░░ 68% [✕]  │
  │ [pdf] relat.pdf   4.0 MB  ✓ pronto     [✕] │
  │ [img] grande.png 22  MB  ⚠ muito grande     │
  └───────────────────────────────────────────┘

Estado
  files: UploadItem[]
  UploadItem {
    id, file: File, previewUrl?: string,
    status: 'queued'|'uploading'|'done'|'error'|'canceled',
    progress: number,        // 0..100
    error?: string, xhr?: XMLHttpRequest
  }
  MAX_CONCURRENT = 3

Contrato de servidor consumido
  POST /api/upload   (multipart/form-data, campo "file")
       -> 201 { id, url } | 413 grande demais | 415 tipo não suportado

Fonte do progresso
  xhr.upload.addEventListener('progress', e => e.loaded / e.total)
```

## Desafios Extras

- Mostre um tempo restante estimado ou a velocidade de upload por arquivo.
- Suporte uploads em blocos (chunked) para arquivos muito grandes com retomada.
- Adicione um controle global de "cancelar tudo" / "repetir todos que falharam".
- Persista os metadados da fila para que um refresh possa oferecer retomar.
- Adicione colar-para-upload a partir da área de transferência.

## Definição de Pronto

- [ ] Arquivos podem ser adicionados via input e arrastar e soltar, e a zona de drop dá feedback visual.
- [ ] Arquivos grandes demais ou de tipo errado são rejeitados com uma mensagem clara por arquivo.
- [ ] As barras de progresso refletem bytes realmente enviados, não uma animação falsa.
- [ ] Não mais que o número configurado de uploads roda concorrentemente.
- [ ] Cancelar e repetir funcionam, e as object URLs são revogadas (sem blobs vazados).

## Armadilhas Comuns

- Usar `fetch` e falsear o progresso — o `fetch` não consegue reportar progresso de upload na maioria dos navegadores; use `XMLHttpRequest`.
- Nunca chamar `URL.revokeObjectURL`, vazando memória conforme o usuário adiciona e remove muitas imagens.
- Disparar todos os uploads de uma vez, saturando a conexão e o servidor.
- Confiar na verificação de tipo no cliente — valide no servidor também; a checagem do cliente é só UX.
- Perder o estado de progresso de um arquivo porque a lista re-renderiza com chaves instáveis.

## Recursos

- [MDN: Usando arquivos de aplicações web](https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications) — a File API de ponta a ponta.
- [MDN: Progresso de upload do XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/progress_event) — progresso real por arquivo.
- [MDN: URL.createObjectURL()](https://developer.mozilla.org/pt-BR/docs/Web/API/URL/createObjectURL_static) — e o correspondente `revokeObjectURL`.
- [MDN: AbortController](https://developer.mozilla.org/pt-BR/docs/Web/API/AbortController) — cancelando trabalho em andamento.
- [web.dev: Arrastar e soltar](https://web.dev/articles/drag-and-drop) — construindo uma zona de drop acessível.
