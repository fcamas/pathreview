# PathReview — Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/97

**Issue title:** Review progress indicator doesn't update in real time during long-running reviews

**Tier:** [x] Tier 3

**Problem summary:**
The review page's progress bar is supposed to reflect the actual status of a
running portfolio review, polled from `GET /reviews/{id}/status` every 5
seconds via the `useReviewStatus` hook. That polling was replaced with a
fixed spinner that never changes, so the progress bar doesn't show real
progress. During long reviews this makes the app look frozen even though
it's still working in the background. The fix affects
`frontend/src/pages/ReviewPage.tsx` and `frontend/src/hooks/useReviewStatus.ts`,
and a successful fix wires the polled status data back into the UI so users
see the review actually advancing in real time instead of a static spinner.

**Branch name:** fix/97-review-progress-realtime

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Reproduction notes (issue #97)

**Steps taken:**
1. Started backing services (`docker compose up -d`) and the app (`make run`), confirmed `http://localhost:5173` and `http://localhost:8000/docs` were reachable.
2. Logged in as `user1@example.com` (seeded test account) and opened the dashboard at `http://localhost:5173/dashboard`.
3. Clicked "Start a New Review", submitted a GitHub username (`octocat`) and a sample resume file, and was redirected to `/reviews/{id}`.
4. Watched the browser's network requests to `GET /api/reviews/{id}/status`, which `useReviewStatus.ts` polls every 3 seconds.

**Observed:**
- Two consecutive polls to `/api/reviews/{id}/status` both returned `{"status":"complete","progress_pct":0}`, meaning even once the review had *fully finished processing*, `progress_pct` was still `0`.
- This confirms the root cause traced in code: `api/routes/reviews.py` (`get_review_status`, ~line 166) returns `getattr(review, "progress_pct", 0)`, but `core/models/review.py`'s `Review` model has no `progress_pct` column, so the `getattr` default (`0`) is always returned. The field is never real data, regardless of pipeline stage.
- Correspondingly, `frontend/src/pages/ReviewPage.tsx` never renders a progress bar or percentage at all. It shows one static "Analyzing your portfolio..." block for the full polling duration, so there's nothing in the UI that could reflect real progress even if the backend sent it.

Full root-cause analysis and fix plan: see `PLAN.md`.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/fcamas/pathreview/commit/d35cca1d23238c43cbd4ed0e113cf4462e7515c5

**Reproduction summary:**
Ran the app locally, logged in, and started a real review through the UI while watching `GET /api/reviews/{id}/status` in the network tab. Two consecutive polls both returned `progress_pct: 0`, even after `status` reached `complete`, confirming `progress_pct` is never populated regardless of pipeline stage.

**PLAN.md link:** https://github.com/fcamas/pathreview/blob/fix/97-review-progress-realtime/PLAN.md

**Walkthrough video (recommended):** Not recorded yet.

**Blockers or open questions:**
Still unsure whether the fix should use fixed milestone percentages (25/50/75/100) at each pipeline stage or something more granular tied to ingestion source count. Plan to confirm against `docs/API.md` before building in Week 9.
