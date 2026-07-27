---
name: Order human translation with Gengo
description: Quote, submit, and retrieve a human translation job through the Gengo API v2.
api: openapi/gengo-openapi.yml
operations: [getServiceLanguagePairs, postServiceQuote, postJobs, getOrder, getJob, updateJob]
---

# Order human translation with Gengo

Use this skill to translate content through Gengo's human translators.

## Authentication
Every call needs `api_key` (public), `ts` (current Unix epoch), and `api_sig`
(HMAC-SHA1 of `ts` keyed by your private key) as query/form params. See
`authentication/gengo-authentication.yml`. Service lookups
(`getServiceLanguagePairs`, `getServiceLanguages`) need only `api_key`.

## Steps
1. **Confirm the language pair** — call `getServiceLanguagePairs` (optionally with
   `lc_src`) to get valid `lc_src`/`lc_tgt`/`tier` combinations and unit prices.
   Never hardcode language codes.
2. **Quote the cost** — call `postServiceQuote` with your jobs map (`data` is the
   flattened JSON of the jobs). This returns credits and unit counts. Confirm the
   account has credits via `getAccountBalance` (error 2700 = not enough credits).
3. **Submit** — call `postJobs` (`POST /translate/jobs`) with the jobs map in the
   `data` form field; set per-job `lc_src`, `lc_tgt`, `tier`, `body_src`, and
   optionally `callback_url` and `auto_approve`. Keep orders 500–2000 words. The
   response returns an `order_id` and job ids.
4. **Track status** — poll `getOrder` (or `getJob`) until jobs reach `reviewable`
   or `approved`, or receive updates on your `callback_url`
   (see `asyncapi/gengo-callbacks-webhooks.yml`). Status flow:
   queued → available → pending → reviewable → approved.
5. **Approve or revise** — when a job is `reviewable`, call `updateJob`
   (`PUT /translate/job/{id}`) with action `approve`, `revise`, or `reject`.
   If you do nothing within 120 hours the job auto-approves.

## Rules
- Requests are `application/x-www-form-urlencoded`; JSON bodies are flattened into
  the single `data` param (compact separators). Max request 100MB.
- Responses are `{opstat, response}` on success or `{opstat:"error", err:{code,msg}}`
  on failure — map the numeric code (see `errors/gengo-error-codes.yml`).
- No idempotency key: error 1754 rejects an identical payload seen within 5 minutes,
  so do not blindly retry a submit — check `getJobs` first.
- Test everything against the sandbox (`http://api.sandbox.gengo.com/v2/`) with
  sandbox keys before going live.
