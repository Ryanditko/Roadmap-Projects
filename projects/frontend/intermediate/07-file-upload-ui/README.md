# File Upload UI (Progress & Preview)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a file-upload interface with drag-and-drop, per-file previews, live progress bars, and the ability to cancel or retry a failed upload. Anyone can wire an `<input type="file">`; the interesting work is everything around it — validating type and size before a byte leaves the browser, tracking real upload progress (which `fetch` alone cannot report), managing a queue of concurrent uploads without saturating the connection, and cleaning up the object URLs you create for previews so the tab does not leak memory. It is a focused study of the File API, upload progress, and resilient async UI.

## Prerequisites

- Comfort with component state and lists in a framework (React, Vue, Svelte, or Angular)
- Familiarity with the File and Blob objects and `FileReader`/`URL.createObjectURL`
- Understanding of `XMLHttpRequest` upload progress events (or the Fetch streaming alternative)
- Basic knowledge of promises and cancellation (`AbortController`)
- Awareness of drag-and-drop events for a drop zone

## Learning Objectives

By the end, you should be able to:

- Accept files via both a file input and a drag-and-drop zone
- Validate file type and size on the client and reject bad files with clear messaging
- Report true upload progress per file using `XMLHttpRequest` progress events
- Manage a queue with a concurrency limit rather than firing all uploads at once
- Cancel an in-flight upload and retry a failed one
- Generate and revoke object URLs for previews to avoid memory leaks

## Functional Requirements

1. Users must add files by clicking an input and by dragging files onto a drop zone that reacts visually.
2. Each file must be validated against an allowed type list and a max size, with a per-file error on rejection.
3. Image files must show a thumbnail preview; non-images must show a type icon and metadata (name, size).
4. Each uploading file must show a live progress bar reflecting actual bytes sent.
5. Uploads must run through a queue with a bounded concurrency (e.g. 3 at a time).
6. A user must be able to cancel an in-flight upload and retry one that failed.
7. Preview object URLs must be revoked when a file is removed or the upload completes.

## Suggested Milestones

1. **Milestone 1 — Select & preview:** Handle input + drop-zone selection and render validated previews.
2. **Milestone 2 — Upload & progress:** Upload each file with a real progress bar via `XMLHttpRequest`.
3. **Milestone 3 — Queue:** Bound concurrency and process the queue as slots free up.
4. **Milestone 4 — Control:** Add cancel, retry, and object-URL cleanup.

## Data & Interface Sketch

```text
Layout
  ┌───────────────────────────────────────────┐
  │   ⬆  Drag files here or [ browse ]         │
  ├───────────────────────────────────────────┤
  │ [img] photo.jpg   1.2 MB  ▓▓▓▓▓░░ 68% [✕]  │
  │ [pdf] report.pdf  4.0 MB  ✓ done       [✕] │
  │ [img] big.png    22  MB  ⚠ too large       │
  └───────────────────────────────────────────┘

State
  files: UploadItem[]
  UploadItem {
    id, file: File, previewUrl?: string,
    status: 'queued'|'uploading'|'done'|'error'|'canceled',
    progress: number,        // 0..100
    error?: string, xhr?: XMLHttpRequest
  }
  MAX_CONCURRENT = 3

Server contract consumed
  POST /api/upload   (multipart/form-data, field "file")
       -> 201 { id, url } | 413 too large | 415 unsupported type

Progress source
  xhr.upload.addEventListener('progress', e => e.loaded / e.total)
```

## Stretch Goals

- Show an estimated time remaining or upload speed per file.
- Support chunked uploads for very large files with resumability.
- Add a global "cancel all" / "retry all failed" control.
- Persist the queue metadata so a refresh can offer to resume.
- Add paste-to-upload from the clipboard.

## Definition of Done

- [ ] Files can be added via input and drag-and-drop, and the drop zone gives visual feedback.
- [ ] Oversized or wrong-type files are rejected with a clear per-file message.
- [ ] Progress bars reflect real bytes sent, not a fake animation.
- [ ] No more than the configured number of uploads run concurrently.
- [ ] Cancel and retry work, and object URLs are revoked (no leaked blobs).

## Common Pitfalls

- Using `fetch` and faking progress — `fetch` cannot report upload progress in most browsers; use `XMLHttpRequest`.
- Never calling `URL.revokeObjectURL`, leaking memory as users add and remove many images.
- Firing every upload at once, saturating the connection and the server.
- Trusting the client-side type check — validate on the server too; the client check is UX only.
- Losing a file's progress state because the list re-renders with unstable keys.

## Resources

- [MDN: Using files from web applications](https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications) — the File API end to end.
- [MDN: XMLHttpRequest upload progress](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/progress_event) — real per-file progress.
- [MDN: URL.createObjectURL()](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL_static) — and the matching `revokeObjectURL`.
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) — cancelling in-flight work.
- [web.dev: Drag and drop](https://web.dev/articles/drag-and-drop) — building an accessible drop zone.
