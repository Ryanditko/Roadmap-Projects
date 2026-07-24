# File Upload API

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 5–8 hours

## Overview

Build an API that accepts file uploads over `multipart/form-data`, stores them safely on local disk, and lets clients list, download, and delete them. Think of the avatar uploader behind a profile page or the attachment box in a support form. The interesting challenges are all about safety: validating type and size, generating filenames that cannot escape your storage directory, and never trusting the name the client sent.

## Prerequisites

- Comfort building HTTP endpoints that return JSON ([Notes API with File Persistence](../04-notes-api-file-persistence/) covers file I/O)
- Understanding of what `multipart/form-data` is and how it differs from a JSON body
- Awareness of MIME types and file extensions
- Familiarity with file-system paths and permissions

## Learning Objectives

By the end, you should be able to:

- Parse a `multipart/form-data` upload and access the file stream and metadata
- Enforce file-type and file-size limits before writing anything to disk
- Generate safe, unique storage names and prevent path-traversal attacks
- Store and serve file metadata separately from the bytes
- Implement download with correct `Content-Type` and `Content-Disposition` headers

## Functional Requirements

1. The system must accept a file via `multipart/form-data` and store it on local disk.
2. The system must reject files over a configured size limit with 413.
3. The system must reject disallowed file types with 415, checking more than just the extension.
4. Stored files must be given a unique, sanitized name that cannot overwrite others or escape the upload directory.
5. The system must return metadata (id, original name, size, type, upload time) on a successful upload.
6. The system must list uploaded files and allow downloading one by its id.
7. The system must allow deleting a file by id and return 404 for an unknown id.

## Suggested Milestones

1. **Milestone 1 — Receive & store:** Accept an upload, write it to disk under a generated name, return metadata.
2. **Milestone 2 — Validate:** Enforce size and type limits, rejecting bad uploads before they touch disk.
3. **Milestone 3 — Serve & manage:** Add list, download (with correct headers), and delete endpoints.

## Data & Interface Sketch

```text
POST   /files    (multipart/form-data, field "file")
                 -> 201 { id, originalName, storedName, size, mimeType, uploadedAt }
                 -> 413 too large | 415 unsupported type
GET    /files             -> 200 [ metadata, ... ]
GET    /files/{id}        -> 200 <bytes> + Content-Disposition | 404
DELETE /files/{id}        -> 204 | 404

storedName = <uuid>.<safe-ext>   (never the raw client filename)
```

## Stretch Goals

- Verify the file's real type by inspecting its magic-number header, not just the extension.
- Require authentication so only logged-in users may upload or delete.
- Generate image thumbnails on upload for image types.
- Add a total-storage quota and reject uploads that would exceed it.

## Definition of Done

- [ ] A valid file uploads, is stored under a generated name, and returns correct metadata.
- [ ] Oversized files return 413 and disallowed types return 415, before any disk write.
- [ ] A crafted filename like `../../etc/passwd` cannot write outside the upload directory (test it).
- [ ] Download returns the original bytes with a sensible filename and content type.
- [ ] Deleting a file removes both the bytes and its metadata; unknown ids return 404.

## Common Pitfalls

- Trusting the client-supplied filename and writing it directly — this enables path traversal and overwrites.
- Checking only the extension for type, which is trivially spoofed; inspect the content too.
- Buffering the entire file in memory, which fails on large uploads — stream to disk.
- Reading the size only after fully receiving the file, so the size limit never protects you.

## Resources

- [MDN: Sending form data (multipart)](https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending_and_retrieving_form_data) — how uploads are encoded on the wire.
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) — the security checklist for this exact project.
- [MDN: Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition) — controlling download filenames.
- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal) — the attack your filename handling must defeat.
