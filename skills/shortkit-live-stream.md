---
name: shortkit-live-stream
description: Create a ShortKit live stream, broadcast to its RTMPS endpoint, end it, and clip the recording.
api: ShortKit API
base_url: https://api.shortkit.dev/v1
operations:
  - createLiveStream        # POST   /live-streams
  - getLiveStream           # GET    /live-streams/{id}
  - endLiveStream           # POST   /live-streams/{id}/end
  - createLiveStreamClips   # POST   /live-streams/{id}/clips
auth: Secret key (Authorization: Bearer sk_live_...).
---

# Broadcast a live stream on ShortKit

Publish a real-time broadcast into the feed; when it ends, the recording stays as on-demand video.

## Steps

1. **Create the stream** — `createLiveStream` (`POST /live-streams`) with optional `title`, `protocol` (`hls` or `webrtc`, set at creation and immutable), `latencyMode` (`low`/`reduced`/`standard`, HLS only), and `reconnectWindow` (0-1800s, default 60).
   The response returns `id`, `playbackId`, and — **only here** — `rtmpUrl` and `streamKey`.

2. **Handle credentials securely** — `rtmpUrl` embeds broadcast credentials. Treat it like an API key: forward only to the broadcaster, never log it, never expose it to viewers or commit it. It is not returned again.

3. **Broadcast** — point the encoder at `rtmpUrl`. The stream `status` moves `idle` -> `active`; while `active` it appears in the feed as `contentType: "live_stream"`. Viewers play it via `playbackId`.

4. **Monitor** — `getLiveStream` (`GET /live-streams/{id}`) for `status`, `startedAt`, and viewer state.

5. **End** — `endLiveStream` (`POST /live-streams/{id}/end`). `status` becomes `ended`, `endedAt` is set, and the recording becomes on-demand video. (HLS streams also auto-end if the broadcaster disconnects past `reconnectWindow`.)

6. **Clip (optional)** — `createLiveStreamClips` (`POST /live-streams/{id}/clips`) to cut shorter clips from the recording; each clip becomes its own Content item.

## Rules & conventions

- Envelope `{ data, meta, errors? }`; branch on `errors[0].code`. See `errors/shortkit-problem-types.yml`.
- Secret key only; 120 req/min sliding window (`429 rate_limited`).
- To address the underlying stream from a feed item, use the item's `liveStreamId`. See `data-model/shortkit-data-model.yml`.
