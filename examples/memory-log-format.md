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
