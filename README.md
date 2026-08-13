# crons

Scheduled jobs that must keep running while the servers they support are off.

## The 60-day trap (why `keepalive.yml` exists)

GitHub disables scheduled workflows after **60 days of repository inactivity**,
and a workflow's own runs do **not** count as activity -- only commits do. A cron
therefore cannot keep itself alive: it goes quiet, the clock runs out, and every
schedule in the repo stops without anything failing loudly.

`keepalive.yml` closes that loop by pushing an empty commit once the last commit
is 50+ days old, resetting the clock with ~10 days to spare. It is repo
infrastructure: any cron added here inherits its protection, so new workflows do
not need to repeat the trick.

Pushes made with `GITHUB_TOKEN` do not trigger workflows, so that commit cannot
cause a run loop.

## If schedules stop firing

GitHub has suspended them for inactivity. The workflows API still reports
`state=active` in that condition, so it is invisible from the CLI -- the only
tell is a banner in the Actions tab. Re-enable without touching any file:

```
gh api -X PUT repos/noamzilo/crons/actions/workflows/<id>/enable
```

Verified on 2026-08-13: after a repo sat dormant 9 months, schedules produced
nothing for 30+ minutes, then fired 10 minutes after that call.

## Jobs

| workflow | what it does |
|---|---|
| `keepalive.yml` | keeps this repo's scheduled workflows alive, forever |
| `supabase-ping.yml` | queries the whatsapp_miner Supabase projects so free-tier ones do not pause after ~7 days idle |
