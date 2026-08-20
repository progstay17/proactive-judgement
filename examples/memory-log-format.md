# Proactive Action Log — format example

Every message the agent sends on its own initiative (not in direct
reply to the user) should be logged in a format your memory system can
retrieve later. This is what makes the skill auditable, and what
prevents the agent from repeating itself across wake cycles.

Adapt the storage mechanism to whatever your harness already uses
(a markdown log, a key-value store, a database row) — the fields below
are what matters, not the format.

## Suggested fields

```
timestamp:      2026-08-17T14:32:00+07:00
bucket:         worth-saying-now | urgent | worth-relaying | social | noise
recipient:      user | <third-party identifier>
gateway:        <which channel was used>
trigger:        <what in memory/state caused this — be specific>
message_sent:   <the actual content, or a summary if long>
certainty:      high | medium | low   (only relevant for third-party sends)
reversibility:  <one line — how bad is it if this was wrong>
```

## Example entries

```
timestamp:      2026-08-17T09:15:00+07:00
bucket:         worth-saying-now
recipient:      user
gateway:        telegram
trigger:        deadline for [project] moved up per last conversation, user hasn't acknowledged
message_sent:   "Heads up — the [project] deadline moved to Thursday, want me to adjust anything?"
certainty:      n/a (recipient is user)
reversibility:  n/a
```

```
timestamp:      2026-08-17T18:40:00+07:00
bucket:         worth-relaying
recipient:      <third party>
gateway:        whatsapp
trigger:        shared plan confirmed by user earlier same day; third party hadn't been looped in yet
message_sent:   "Hey, [user] wanted me to pass along that [plan detail] — let me know if that works"
certainty:      high — source was user's direct confirmation same day
reversibility:  low stakes, easily clarified if wrong
```

## Why this matters

Without a log like this, the agent has no way to know it already sent
something, and the user has no way to review what the agent said on
their behalf. Treat this log as required, not optional — `SKILL.md`
Step 5 makes this explicit.

---

## The other layer: shared short-term log (Step 6)

The log above is the durable, curated record — you write to it when
something actually gets sent. That's a different thing from the
short-term log in `SKILL.md` Step 6, which is a rolling, unfiltered
window (roughly 24–48 hours) of *everything* that happened, across
every gateway and session — not just the proactive sends. Its job is
to answer "what just happened, anywhere" fast, before deciding whether
to act.

```
timestamp:  2026-08-19T07:59:00+07:00
gateway:    whatsapp
direction:  inbound
summary:    user sent an unrelated message — confirms user is active
```

```
timestamp:  2026-08-19T08:00:00+07:00
gateway:    telegram
direction:  scheduled-check
summary:    watchdog tick fired; skipped — user was active 1 min ago per short-term log
```

Keep entries here cheap to write and cheap to read — every wake cycle
reads this before anything else, on any gateway. Once something rolls
past the window, summarize anything that matters into the permanent
log above and let the raw entry drop.
