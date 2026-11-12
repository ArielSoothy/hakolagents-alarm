# hakolagents-alarm

An outside alarm for hakolagents.com that does not depend on the owner's Mac.
Every hour (`.github/workflows/outside-alarm.yml`) it reads the public page `https://hakolagents.com/api/callit/health/summary` and sends a Telegram message when the site is unreachable or reports RED, the robot or the Mac has gone quiet, or a scheduled job is overdue.
Its health check only reads that public page and this workflow's own run history (built-in `GITHUB_TOKEN`, `actions: read`); Telegram uses `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`. The optional cloud helper uses its own per-routine secret below.
Run it by hand with `test_alert` ticked to send one test message.
The Mac heartbeat in the main repo watches this workflow and warns when it has not run for 3 hours.
It pages once per problem (plus a reminder every 6 hours while it lasts) and sends one "recovered" message when the check is OK again.
GitHub turns off scheduled workflows in public repos after 60 days without repository activity: push a commit (any small change) before about 2027-01-05, and again within every 60 days after that.

## Cloud helper (T-309)

After a scheduled alarm page has reached Telegram, the alarm can wake a Claude cloud routine to diagnose and report in the Claude Doc 'HaKol Agents — daily operator log'. It fires when the Mac is silent or the site is unreachable, or on a six-hour reminder for a RED site. v1 only diagnoses and notifies; it never changes production. Manual health checks, recovery messages and suppressed repeat pages never fire it.

The operator creates the routine from `docs/ops/cloud-fallback-routine-prompt.md` in the private repository, then configures these settings in this alarm repository:

- Repository variable `CLAUDE_FALLBACK_ROUTINE`: the routine's `trig_` id.
- Repository secret `CLAUDE_FALLBACK_TOKEN`: the per-routine token Ariel generates in `claude.ai/code/routines` → routine → Add another trigger → API → Generate token. The token is shown once; store it directly as the secret and never put it in a file or log.

Actions → HaKol outside alarm → Run workflow → tick `test_fallback` to fire once with a TEST payload, without a health read. Leave `test_alert` unticked: it takes precedence if both switches are ticked. A successful test sends 'TEST — cloud helper started' and the routine should write a test receipt and push 'HaKol cloud helper test OK'.

With either setting missing, the alarm works exactly as before and sends no extra Telegram. Pause the routine or delete its token secret to stop the helper. A configured but refused fire sends a short 'could not start' Telegram after the alarm page; follow RUNBOOK-jobs.md 'Cloud fallback' in the private repo.
