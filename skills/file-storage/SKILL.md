---
name: file-storage
description: Upload, download, and manage files with MoonDB storage. Use when implementing file uploads, avatars, or media.
---

# File Storage

## When to use

- Adding file uploads (avatars, images, documents) to an app
- Implementing media galleries or file attachments
- Connecting uploaded files to database records

## Prerequisites

Add a `file` type column to your schema:

```json
{
  "tables": {
    "posts": {
      "columns": {
        "title": "string required",
        "cover_image": "file",
        "attachment": { "type": "file", "accept": "application/pdf", "max_size_mb": 5 }
      }
    }
  }
}
```

File column options:
- `accept` — MIME type filter (e.g., "image/*", "application/pdf")
- `max_size_mb` — max file size in megabytes (default: 10 MB)

## Upload a file

```
POST /p/{project_id}/storage/upload
Authorization: Bearer {token}
Content-Type: multipart/form-data

file: (binary)
table: posts
column: cover_image
row_id: (optional, validates against schema)
```

Response:
```json
{
  "data": {
    "id": "file_...",
    "url": "https://api.moondb.ai/p/{project_id}/storage/file_...",
    "size": 245760
  }
}
```

## Link file to a record

After uploading, store the URL in the record:

```
PATCH /p/{project_id}/api/posts/{post_id}
Content-Type: application/json

{ "cover_image": "https://api.moondb.ai/p/{project_id}/storage/file_..." }
```

## Download a file

```
GET /p/{project_id}/storage/{file_id}
```

## Delete a file

```
DELETE /p/{project_id}/storage/{file_id}
Authorization: Bearer {token}
```

## JavaScript example

```javascript
async function uploadFile(file, table, column) {
  const form = new FormData();
  form.append('file', file);
  form.append('table', table);
  form.append('column', column);

  const res = await fetch(`${API_URL}/storage/upload`, {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}` },
    body: form,
  });
  return res.json();
}
```
