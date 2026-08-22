# 9. Automation & Scheduling

You need two different kinds of "make this happen without me": **start on
boot** (so a machine restart doesn't take your service down until you
notice) and **run on a schedule** (daily digests, expiry reminders,
cleanup). Neither needs a real job-queue system at solo-project scale.

## Start-on-boot

A small script that, for each process your stack needs:
1. Checks if it's already running (by port-listening state for network
   services, by process name otherwise).
2. If not, starts it — hidden/background for things that don't need a
   visible window, `-NoExit` (or your shell's equivalent) for anything you
   might want to glance at the live log of.
3. Optionally waits a moment and does one health check at the end, so you
   get immediate confirmation everything actually came up rather than
   three silently-failed background launches.

Register that script as a startup task. On Windows, `Register-ScheduledTask`
with a startup trigger; on Linux, a user-level `systemd` service or a cron
`@reboot` entry; on macOS, a `launchd` agent. All equivalent in spirit —
"run this on boot, restart it if it dies."

**Idempotency matters more than elegance here**: running the start script
twice (once by the scheduler, once by you double-clicking a shortcut)
should be a safe no-op for anything already running, not a duplicate
process or an error.

## A visible-feedback gotcha

If your start script only *does* something when a process isn't already
running, and does nothing (or something invisible) when everything's
already up, manually re-running it later — after the scheduler already
started everything on boot — looks like "nothing happened, is this broken?"
even though it worked exactly as designed. If you want it to be
reassuring on every run, not just the first: always end the script with
something visibly confirming state, even when there was nothing to start.
A good default: open (or bring to front) whatever dashboard/admin view lets
you see the system is alive and doing something (current signups, active
connections, last request time) — not just a health-check JSON blob that
flashes and disappears.

## Recurring jobs (daily reminders, cleanup, digests)

A plain script + a scheduled trigger, same idea as start-on-boot but with
a time-based trigger instead of a boot trigger:

```text
Windows: New-ScheduledTaskTrigger -Daily -At 9:00AM
Linux/macOS: a cron line, e.g.  0 9 * * *  /path/to/script
```

Design the script itself to be **safe to run more than once on the same
day** — register a "already did this today" marker per unit of work (e.g.
a `sent_at` timestamp column on the row the job acts on) and check it
before acting, rather than relying on the scheduler to never double-fire
(it will, eventually — a manual re-run while debugging, a missed run that
gets caught up, a scheduler quirk). This turns "did this actually only run
once" from a scheduler-reliability question into a code-correctness
question you can unit test.

## What actually needs administrator/root privileges

Less than intuition suggests:

- A scheduled task that runs **only while you're logged in, on your own
  schedule** (daily at a fixed time, boot-of-your-session) — typically does
  **not** need elevated privileges on Windows; a per-user task registration
  is enough.
- A task that must run **before any user logs in**, or **as a different
  system account**, generally does need elevation.

If your first attempt at registering a scheduled task fails with a
permissions error, check whether you actually need the elevated variant
before reaching for "run as administrator" — the extra prompt is friction
your future self (or a teammate) will hit every time they need to touch it.

## Logs

Redirect stdout/stderr from every backgrounded process to a file under a
`logs/` directory (gitignored — these will contain request data, error
traces, sometimes emails/IPs). When a background start silently fails,
this is the only place you'll find out why — always create the log
directory as part of the start script (`New-Item -ItemType Directory
-Force`) so a missing directory isn't itself the reason nothing got logged.
