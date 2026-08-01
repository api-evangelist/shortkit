---
name: shortkit-upload-video
description: Upload a video to ShortKit, complete the direct signed-URL transfer, and confirm it is ready to play.
api: ShortKit API
base_url: https://api.shortkit.dev/v1
operations:
  - uploadContent   # POST /content/upload
  - getContent      # GET  /content/{id}
auth: Secret key (Authorization: Bearer sk_live_...) — or a publishable key (X-API-Key pk_live_...) for client-side uploads.
---

# Upload a video to ShortKit

Get a video into a ShortKit feed. ShortKit never proxies the file — you receive a signed URL and upload the bytes directly to cloud storage.

## Steps

1. **Create the upload** — `uploadContent` (`POST /content/upload`) with a JSON body:
   `contentType: "video"` (required), `title` (required, <=200 chars), and optional `section`, `tags`, `description`, `author`, `protected`, `publishAt`, `expiresAt`, `callbackUrl`.
   For iOS-recorded HEVC video set `origin: "ios_upload"` for color-accurate thumbnails.
   The response `data` includes the new content `id`, `uploadStatus: "pending"`, and a signed `uploadUrl`.

2. **Upload the file** — `PUT` the raw video bytes to the returned `uploadUrl` with the correct `Content-Type` (e.g. `video/mp4`). This goes to cloud storage, not the API.

3. **Track processing** — `uploadStatus` transitions `pending` -> `processing` -> `ready`. Either:
   - poll `getContent` (`GET /content/{id}`) until `uploadStatus == "ready"`, or
   - supply `callbackUrl` at step 1 and handle the `content.imported` webhook (status `processing`, then `ready`), and `content.import_failed` (status `error`). Respond `2xx` to each. See `asyncapi/shortkit-webhooks.yml`.

## Rules & conventions

- Responses use the envelope `{ data, meta, errors? }`; capture `meta.request_id` for support.
- Handle errors by `errors[0].code` (`invalid_request`, `invalid_api_key`, ...), not HTTP status. See `errors/shortkit-problem-types.yml`.
- Secret keys are server-side only; never embed them in client apps. Use a publishable key for in-app uploads.
- Secret-key requests are limited to 120/min (sliding window); back off on `429 rate_limited`.
