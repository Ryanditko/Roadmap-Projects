# Design a File Storage System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a service that lets users upload, store, and download files — a simplified Dropbox or S3. The interesting tension is that files are large and opaque blobs, while everything you need to search and list is small metadata. A good design separates the two: blobs go to object storage, metadata goes to a database. You'll reason about durability, how large uploads are handled, and how a download stays fast. Deliver a design document that explains the split, the API, and how files survive hardware failure.

## Prerequisites

- Understanding of the difference between a file's bytes and its metadata
- Awareness of what object storage (like S3) provides versus a database
- Familiarity with HTTP upload/download and content types
- Comfort reasoning about redundancy and copies of data

## Learning Objectives

By the end, you should be able to:

- Separate blob storage from metadata storage and justify why
- Reason about durability through replication
- Design an upload flow for large files (chunking, presigned URLs)
- Estimate storage growth and bandwidth
- State a trade-off between simplicity and durability

## Requirements & Constraints

1. Upload a file and get back a stable identifier or URL.
2. Download a file by its identifier.
3. List a user's files with their metadata (name, size, upload time).
4. Files must survive the loss of a single storage node (durability).
5. Support files up to a defined size limit; handle large ones gracefully.
6. Estimate scale: 1M users, average 20 files each at 2 MB — roughly 40 TB total.

## Suggested Approach

1. Draw the boundary: blobs to object storage, metadata to a database.
2. Do the math: 1M × 20 × 2 MB = 40 TB; plan replication factor (e.g., ×3 = 120 TB raw).
3. Design the upload path — for large files, describe chunked upload or a presigned URL so bytes skip your app server.
4. Design the download path and consider a CDN for frequently accessed files.
5. Describe how replication delivers durability and what happens when a node fails.

## Architecture Sketch

```text
Client ── request upload ──> [ App ] -- presigned URL --> Client
Client ── PUT bytes ───────> [ Object Store (replicated ×3) ]
Client ── GET /files/{id} ─> [ App ] -> metadata + [ CDN / Object Store ]

Core API
  POST /files            { name, size }   -> 201 { fileId, uploadUrl }
  PUT  <uploadUrl>       (raw bytes)       -> 200
  GET  /files/{id}                         -> 302 to blob | stream
  GET  /files            ?owner=me         -> [ { fileId, name, size, createdAt } ]

Data model
  files: file_id (PK) | owner_id | name | size | content_type | blob_key | created_at
  blob:  stored in object store keyed by blob_key, replicated across nodes
```

## Deep-Dive Topics

- **Durability:** replication factor vs. erasure coding, and what "eleven nines" actually means.
- **Large uploads:** chunking, resumable uploads, and why bytes should bypass the app server.
- **Deduplication:** content-hash keys so identical files share one blob (a stretch).

## Deliverables

- An architecture diagram showing the blob/metadata split and upload/download paths.
- The core API contract for upload, download, and listing.
- A data model separating file metadata from blob storage.
- One trade-off written up: e.g., storing blobs in the database (simple, one system) vs. dedicated object storage (scales, durable, but two systems to coordinate).

## Common Pitfalls

- Streaming every byte through the app server, making it the bottleneck.
- Storing large blobs in a relational database and hitting size and performance walls.
- Treating a single copy as safe — durability requires redundancy.
- Forgetting that metadata queries (list, search) need their own indexed store.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — object storage and durability patterns.
- [AWS S3: How it works](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) — the reference object storage service.
- [AWS: Uploading and copying objects using multipart upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html) — how large uploads are chunked.
- [Wikipedia: Erasure code](https://en.wikipedia.org/wiki/Erasure_code) — an alternative to replication for durability.
