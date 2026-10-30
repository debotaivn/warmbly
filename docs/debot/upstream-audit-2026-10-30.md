# Warmbly upstream backport audit — 2026-10-30

## Baselines
- Debot fork `debotaivn/warmbly` main: `c8df7fb7cb7f8846cab896b7adc5b7fbee665f05` (2026-09-02).
- Warmbly upstream `warmbly/warmbly` main checked: `5279b8119e14f9c8bc1f2090fda9f3536dd4a07a` (2026-10-30).
- GitHub compare reports **2,099 upstream commits ahead**. Do NOT merge all blindly: modules, schemas, migrations, auth, and deployment topology changed.

## Backported in this candidate release
1. [Issue #340](https://github.com/warmbly/warmbly/issues/340): false no-sender preflight and incorrect zero-step launch label. Added shared `ResolveCampaignSenderPool`, updated preflight/scheduler, step API read, DB readiness and Advisor fallback, with unit tests.
2. [PR #1048](https://github.com/warmbly/warmbly/pull/1048): calendar-aware campaign send windows over daylight-saving transitions. Backported date walking, skipped-hour handling, fail-closed slot behavior, plus upstream DST sweep tests.

## Further upstream fixes — require separate compatibility / security review
| Upstream | Change | Why not blindly merge this release |
|---|---|---|
| [PR #1084](https://github.com/warmbly/warmbly/pull/1084) | Preserve campaign send attempts during mailbox reconnect retries | Needs comparison of consumer send-result event semantics and DB lifecycle with Debot |
| [PR #1091](https://github.com/warmbly/warmbly/pull/1091) | Isolate unconfirmed sends to prevent duplicate delivery | 34 files + database migration 000331; high-impact state transition |
| [PR #1018](https://github.com/warmbly/warmbly/pull/1018) | Recover parked campaign/warmup tasks after schedule changes | 27 files across scheduler, consumer, models, repo |
| [PR #1077](https://github.com/warmbly/warmbly/pull/1077) | Enforce SMTP authentication policy for hosted relays | 13 files touching SMTP provider behavior and limits |
| [PR #1056](https://github.com/warmbly/warmbly/pull/1056) | Workspace isolation and sensitive-route re-auth | 174 files; requires independent security regression |
| [PR #1054](https://github.com/warmbly/warmbly/pull/1054) | Dependency security advisories | 17 dependency/lock files across Go, Rust, JS, Elixir |
| [PR #1064](https://github.com/warmbly/warmbly/pull/1064) | Patched realtime JOSE | Requires Elixir lock/dependency compatibility |
| [PR #1063](https://github.com/warmbly/warmbly/pull/1063) | Patched tracking Tokio | Requires Rust lock/dependency compatibility |
| [PR #1030](https://github.com/warmbly/warmbly/pull/1030) | KMS-wrapped mailbox credentials | 40 files and key-management changes; migration/rekey plan required |

## Production release gate
- [ ] Run `go test -race ./...`, Go lint and Go build.
- [ ] Run `pnpm lint`, `pnpm typecheck`, `pnpm test:run`, `pnpm build` in `web`.
- [ ] Confirm the preflight is clear when a single directly selected Gmail is active, and fails when no active mailbox exists.
- [ ] Verify actual step count on the Launch modal.
- [ ] Test one synthetic recipient with tracking on/off, confirm no unintended duplicate sends.
- [ ] Validate Berlin/Budapest send windows around 2026-10-25 and other DST boundaries.
- [ ] Snapshot DB and Docker images; stage on VPS, then promote after verification.
- [ ] Keep current production unchanged until every gate is passed.

These fixes have been committed to the test branch; no CI pass, deploy, or production success is claimed in this document.
