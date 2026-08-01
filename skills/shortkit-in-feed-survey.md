---
name: shortkit-in-feed-survey
description: Create an in-feed ShortKit survey, activate it, collect responses, and read aggregated results.
api: ShortKit API
base_url: https://api.shortkit.dev/v1
operations:
  - createSurvey          # POST /surveys
  - updateSurvey          # PUT  /surveys/{id}
  - getSurvey             # GET  /surveys/{id}
  - submitSurveyResponse  # POST /surveys/{id}/responses
auth: Secret key (Bearer) for management; publishable key (X-API-Key) for response submission.
---

# Run an in-feed survey on ShortKit

Surveys are short polls the SDK renders between content items. You define the question, options, and placement rules; the SDK handles display and collection.

## Steps

1. **Create** — `createSurvey` (`POST /surveys`) with `question` (shown to users), `options[]` (`text`, `position`), optional internal `title`, `priority`, and `autoAdvanceDelay`. New surveys start in `status: "draft"`.

2. **Activate** — `updateSurvey` (`PUT /surveys/{id}`) to set `status: "active"`. Only `active` surveys are injected into the feed. Statuses: `draft`, `active`, `paused`, `archived`.

3. **Placement rules** — configure `placementRules` (e.g. `min_videos_before` with `{ n: 3 }`, or `once_per_user`) to control when the survey appears.

4. **Collect responses** — the SDK submits `submitSurveyResponse` (`POST /surveys/{id}/responses`) with an `optionId` using the publishable key. A duplicate submission returns `409 conflict`.

5. **Read results** — `getSurvey` (`GET /surveys/{id}`) returns `options[].responseCount` and `totalResponses`.

## Rules & conventions

- Envelope `{ data, meta, errors? }`; handle `errors[0].code` (`conflict`, `not_found`, ...). See `errors/shortkit-problem-types.yml`.
- No idempotency key is available; the `409 conflict` on duplicate responses is the dedup signal. See `conventions/shortkit-conventions.yml`.
- Secret-key management calls are limited to 120/min (sliding window).
