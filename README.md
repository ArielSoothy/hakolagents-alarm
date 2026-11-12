# hakolagents-alarm

An outside alarm for hakolagents.com that does not depend on the owner's Mac.
Every hour (`.github/workflows/outside-alarm.yml`) it reads the public page `https://hakolagents.com/api/callit/health/summary` and sends a Telegram message when the site is unreachable or reports RED, the robot or the Mac has gone quiet, or a scheduled job is overdue.
It only reads that public page and this workflow's own run history (built-in `GITHUB_TOKEN`, `actions: read`): no logins, no private data, and the only secrets are `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`.
Run it by hand with `test_alert` ticked to send one test message.
The Mac heartbeat in the main repo watches this workflow and warns when it has not run for 3 hours.
It pages once per problem (plus a reminder every 6 hours while it lasts) and sends one "recovered" message when the check is OK again.
GitHub turns off scheduled workflows in public repos after 60 days without repository activity: push a commit (any small change) before about 2027-01-05, and again within every 60 days after that.
