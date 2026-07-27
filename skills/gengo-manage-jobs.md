---
name: Monitor and manage Gengo translation jobs
description: List, inspect, comment on, and review Gengo jobs and their revisions.
api: openapi/gengo-openapi.yml
operations: [getJobs, getJobsByIds, getJob, getJobRevisions, getJobRevision, postJobComment, getJobComments, getJobFeedback, updateJob]
---

# Monitor and manage Gengo translation jobs

Use this skill to track in-flight jobs and communicate with translators.

## Authentication
All operations here are authenticated: send `api_key`, `ts`, and `api_sig`
(HMAC-SHA1 of `ts`). See `authentication/gengo-authentication.yml`.

## Steps
1. **List recent jobs** — call `getJobs` (`GET /translate/jobs`) filtered by
   `status`, `timestamp_after`, and `count`. Fetch specific ones with
   `getJobsByIds` (comma-separated ids) or a single job with `getJob`.
2. **Inspect revisions** — call `getJobRevisions` to list revision resources, then
   `getJobRevision` for a specific `rev_id`. Revisions are created each time a
   translator revises.
3. **Communicate** — post instructions with `postJobComment`
   (`POST /translate/job/{id}/comment`, `data` = `{"body":"..."}`) and read the
   thread with `getJobComments`.
4. **Review** — read customer feedback with `getJobFeedback`; when a job is
   `reviewable`, drive it to a terminal state with `updateJob` (approve / revise /
   reject).

## Rules
- Job access is scoped to the authenticated account (error 2100 = unauthorized job
  access; 2250 = job not reviewable; 2252 = cannot cancel in current state).
- The translated content is only present in `reviewable` and `approved` statuses.
- Comment/threads and payloads over 60KB will not be delivered via callbacks — fetch
  large content with `getJob`/`getOrder` instead.
- Map every error via the numeric `err.code` (`errors/gengo-error-codes.yml`).
