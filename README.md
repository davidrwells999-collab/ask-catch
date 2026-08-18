# Askable Opportunity Alerts

Polls Askable's GraphQL API every 5 minutes and pushes a phone notification via
[ntfy.sh](https://ntfy.sh) when a new opportunity appears.

## ⚠️ The one real limitation: the auth token expires every 24 hours

The bearer token you captured from DevTools is a short-lived JWT — it expires
24 hours after it's issued. This script has no way to refresh it on its own
(that would need Askable's login/refresh-token flow, which is a bigger build).

**Practical workaround:** once a day, re-open DevTools on my.askable.com,
grab the fresh `Authorization` header value (same steps as before), and
update the `ASKABLE_TOKEN` secret in GitHub. Takes about 30 seconds.

The script will message you via ntfy if the token has expired and the check
starts failing (401/403), so you'll know when it needs refreshing rather than
silently missing opportunities.

If this becomes annoying, the proper fix later is implementing the OAuth
refresh-token flow — worth doing only if the daily copy-paste gets old.

## Setup

1. **Create a GitHub repo** (private is fine) and push these files to it.

2. **Set up ntfy:**
   - Install the [ntfy app](https://ntfy.sh/) on your phone (iOS/Android)
   - Pick a random, hard-to-guess topic name (e.g. `dave-askable-x7k2p9`) —
     anyone who knows your topic name can send you notifications or read them,
     since topics aren't private by default
   - Subscribe to that topic in the app

3. **Add repo secrets** (Settings → Secrets and variables → Actions):
   - `ASKABLE_TOKEN` — the full `Authorization` header value from DevTools
     (starts with `eyJ...`)
   - `ASKABLE_USER_ID` — your `_user_id`, visible in the request payload's
     `variables.search._user_id` (e.g. `69f8de966781114b41ebc177`)
   - `NTFY_TOPIC` — the topic name you picked above

4. **Enable Actions** on the repo if prompted, then trigger it once manually
   (Actions tab → Check Askable Opportunities → Run workflow) to confirm it
   works before waiting for the schedule.

## Notes

- First run will notify you about *every* currently-live opportunity (nothing
  is "seen" yet). After that, only genuinely new ones trigger a push.
- `seen-opportunities.json` is committed back to the repo after each run to
  persist state between workflow executions.
- GitHub's cron schedule isn't exact under load — 5 minutes can occasionally
  slip to 7–10. If that's too loose, GitHub Actions doesn't support sub-5-minute
  cron; a self-hosted runner or an external cron service (e.g. cron-job.org
  hitting a `workflow_dispatch` webhook) would be the way to tighten it.
