---
name: Verify a Lemin CAPTCHA answer
description: Confirm server-side that a website visitor solved a Lemin gamified CAPTCHA before trusting their request.
api: openapi/capy-inc-captcha-openapi.yml
operations: [validateCroppedCaptcha]
---

# Verify a Lemin CAPTCHA answer

Use this skill to validate the answer a visitor produced for a Lemin cropped
(gamified puzzle) CAPTCHA. Verification is a single server-side call.

## Prerequisites

- Your account **private_key** (secret) from the Lemin Dashboard. Keep it on the
  server — never send it from the browser.
- The **challenge_id** and encrypted **answer** produced by the client-side
  widget for this session.

## Steps

1. Collect `challenge_id` and `answer` from the front end after the visitor
   completes the puzzle.
2. Call `validateCroppedCaptcha` — `POST https://api.leminnow.com/captcha/v1/cropped/validate`
   with `Content-Type: application/json` and a body of:
   `{ "private_key": "<secret>", "challenge_id": "<id>", "answer": "<encrypted>" }`.
3. The endpoint always returns HTTP 200. Read the JSON body:
   - `success: true` (with `message` and `code` null) → the visitor is human;
     proceed with the protected action.
   - `success: false` → verification failed; inspect `code`.

## Handling failures (code field)

- `incorrect_answer` — re-present the puzzle and let the visitor retry.
- `challenge_is_not_active` — the challenge expired or was already used; issue a
  fresh one. Do not reuse a `challenge_id`.
- `invalid_challenge_id` / `invalid_parameters` — check the values passed from
  the widget and that the body is valid JSON.
- `invalid_private_key` — the account key is wrong; fix server configuration.

## Rules

- Always verify server-side. Never expose `private_key` to the client.
- A `challenge_id` is single-use; verify promptly after the visitor solves it.
- See errors/capy-inc-error-codes.yml and conventions/capy-inc-conventions.yml
  for the full envelope and semantics.
