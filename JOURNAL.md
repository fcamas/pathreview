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

**Cohort ledger:** [ ] Issue added to cohort ledger
