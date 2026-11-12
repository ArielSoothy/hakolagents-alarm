# hakolagents-alarm

An outside alarm for hakolagents.com that does not depend on the owner's Mac.
Every hour (`.github/workflows/outside-alarm.yml`) it reads the public page `https://hakolagents.com/api/callit/health/summary` and sends a Telegram message when the site is unreachable or reports RED, the robot or the Mac has gone quiet, or a scheduled job is overdue.
Its `check` job only reads that public page and this workflow's own run history (built-in `GITHUB_TOKEN`, only `actions: read`); Telegram uses `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`. A separate `wake` job has only `contents: write` to request the cloud helper through a release.
Run it by hand with `test_alert` ticked to send one test message.
The Mac heartbeat in the main repo watches this workflow and warns when it has not run for 3 hours.
It pages once per problem (plus a reminder every 6 hours while it lasts) and sends one "recovered" message when the check is OK again.
GitHub turns off scheduled workflows in public repos after 60 days without repository activity: push a commit (any small change) before about 2027-01-05, and again within every 60 days after that.

## Cloud helper (T-309)

After a scheduled alarm page has reached Telegram, the alarm can request a Claude cloud routine to diagnose and report as an issue in this repository (Claude-Docs is not available to cloud runs). It requests a wake when the Mac is silent or the site is unreachable, or on a six-hour reminder for a RED site. v1 only diagnoses and notifies; it never changes production. Manual health checks, recovery messages and suppressed repeat pages never request a wake.

The operator creates the routine from `docs/ops/cloud-fallback-routine-prompt.md` in the private repository, with GitHub trigger `release.published` on `ArielSoothy/hakolagents-alarm` and BOTH `ArielSoothy/hakolagents` and `ArielSoothy/hakolagents-alarm` as sources. The Claude GitHub App is already installed for all repos. No personal or routine API token is needed. The built-in `GITHUB_TOKEN` creates one prerelease, tagged `fallback-<run>-<epoch>-<reason>`; there is one POST, a 15-second timeout, and no retry.

Telegram says 'Cloud helper wake requested (<reason>)' and points to the doc report due in ~15 minutes. A release only proves the request; the helper's own doc entry (naming the tag) and required push confirm it ran. No report means it did not start; follow RUNBOOK-jobs.md 'Cloud fallback' in the private repo by hand. A release failure sends 'Cloud helper could not be woken (<status>)' and fails the wake job.

Actions → HaKol outside alarm → Run workflow → tick `test_fallback` to request one TEST release, without a health read. Leave `test_alert` unticked: it takes precedence if both switches are ticked. A successful request sends 'TEST — cloud helper wake requested'; the routine should write a tagged test receipt and push 'HaKol cloud helper test OK'. The test fails if release creation or its Telegram fails. If the end-to-end test does not start a routine session, drop this route and keep the alarm Telegram-only; never substitute a personal token.

Switch off by removing the routine's GitHub trigger or pausing the routine. The alarm still pages Telegram; a later release request does not prove the paused helper started. After creation, best-effort cleanup lists up to 100 releases, keeps the newest 20 `fallback-*` releases and deletes older fallback releases AND their tags. Non-fallback releases and tags are never touched; cleanup failures only log `OUTSIDE_ALARM_WAKE_CLEANUP_FAILED` and do not change the result.

Secrets: `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`; GitHub supplies its job-scoped `GITHUB_TOKEN`. The check job cannot write repository contents, and the wake job cannot read Actions history. There are no external `uses:` actions.

Started 2026-11-12. Owner tasks live in the main project's TODO (T-324: keep this repo active).
