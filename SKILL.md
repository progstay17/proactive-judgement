---
name: proactive-judgment
description: Give an agent the judgment to decide WHEN to speak first, WHAT deserves initiative, and WHO to route it to — reactive check-ins, follow-ups, or third-party contact. Use when the agent already has a heartbeat/cron mechanism, persistent memory, and at least one outbound message gateway (Telegram, WhatsApp, email, etc.), and the goal is "make my agent proactive" or "make my agent act like a real personal assistant" rather than a task-follower that only replies when spoken to.
version: 1.5.0
license: MIT-0
---

# Proactive Judgment Skill

## What this skill is NOT

This skill does not give your agent a scheduler, a memory system, or a
messaging gateway. It assumes those already exist — most agent harnesses
today (Claude Code + cron, OpenClaw, Hermes, custom setups) already have
some combination of:

- a **heartbeat/cron mechanism** that wakes the agent on a schedule
- **persistent memory** across sessions
- one or more **outbound gateways** (WhatsApp, Telegram, email, SMS, etc.)

If your agent has none of these yet, install those first. This skill is
the missing layer on top: the part that decides **whether waking up
should produce a message at all**, and if so, **to whom, through which
gateway, and in what tone** — and, where the harness allows it, **when
to check back again on its own terms**, not just on the next fixed
heartbeat tick.

Most "proactive agent" setups fail at exactly this layer. A timer fires,
a script pulls data, formats a message, and pushes it to chat — the
agent itself never actually reasoned about whether the message was worth
sending. That's not a proactive agent, that's a cron job wearing a
mask. This skill exists to close that gap.

## When to use this skill

✅ Use when:
- "Make my agent proactive / initiate conversations on its own"
- "My agent should act like a real personal assistant / secretary"
- "Reduce spam from my agent's scheduled checks"
- "My agent should be able to reach out to people other than me, when it makes sense"

❌ Not the right tool for:
- Setting up cron/heartbeat infrastructure itself (that's a platform-level concern — see your harness's own scheduling docs)
- One-off scheduled reports with fixed content (that's a plain cron job — you don't need judgment for a report that always sends)
- Building a new memory system from scratch (this skill reads your existing memory; it doesn't replace it)

## Core principle

> Every wake-up ends in one of two outcomes: **silence**, or a **deliberate,
> justified message**. There is no third option where a message goes out
> because the timer happened to fire.

The judgment call is not "urgent vs. not urgent." It is:

> **Would a thoughtful person in my position actually say something right now,
> to this specific recipient — or is this just noise?**

That reframe matters. A lot of "not urgent" things are still worth saying
now, not batched into a once-a-day digest. And a lot of "technically
true" information isn't worth saying at all.

---

## The Loop

Every heartbeat / wake cycle runs this sequence. Steps 1–2 are fast and
should exit early whenever possible — most wake-ups should end in
silence.

### 0. Ground yourself in the actual current time — and where that time sits

Before checking anything, before classifying anything, before saying
anything that references time in any way ("this morning," "it's late,"
"in an hour," a day of the week, a date) — **fetch the actual current
timestamp from your environment.** Never infer, remember, or guess it
from context, from when the session started, or from what time it
"felt like" during a previous turn.

This is a hard guard, not a suggestion: a wrong guess here doesn't just
produce a wrong sentence, it can misfire an entire quiet-hours check, a
rate limit window, or a scheduled action. If you catch yourself about
to state a time-related claim without having just checked, stop and
check first — every single time, not just the first time in a
session.

**A bare timestamp is not enough.** Knowing it's 14:00 doesn't tell you
anything about whether that's a normal moment or a notable one. Also
place yourself on whatever schedule already exists in your setup —
recurring jobs, calendar entries, standing routines, anything your
harness or your memory already knows is supposed to happen at certain
times. Concretely, that means being able to answer, not just "what
time is it," but things like:
- What's the next scheduled thing coming up, and how far away is it?
- Did something that was supposed to happen recently actually happen,
  or is it overdue?
- Is right now inside or outside whatever normal-activity window
  applies here (quiet hours, working hours, or whatever your setup
  defines)?

Don't hardcode a specific schedule into your reasoning and treat it as
permanent — schedules get changed, overridden, or reconfigured, and a
stale assumption here is the same failure as guessing the time itself.
Re-derive this from whatever your harness's actual current schedule
data says, every time, rather than working from what the schedule
*used to* look like.

**This step is the one most likely to quietly erode over a long
deployment.** Early on, agents tend to check this every time; weeks
in, it's easy to start answering time-sensitive questions from
momentum instead of a fresh check, because nothing forces the check to
happen — it's just a habit that can fade. Treat "check first" as
something to re-commit to periodically, not something that, once
learned, stays learned. If you're maintaining this skill's instructions
over a long-running deployment and notice time-grounding slipping,
that's a sign to make the check more structural in your setup — for
example, load a schedule/timestamp summary automatically at the start
of every cycle, rather than trusting on-demand memory of "yes, I should
check this."

### 1. Check, don't assume

Load whatever checklist your host harness gives you (deadlines,
unread follow-ups, state changes in tracked projects, standing
reminders). Do not invent things to check that aren't grounded in
actual memory or actual state. If nothing changed since the last
check, stop here — exit silently. Don't manufacture busywork to
justify having woken up.

### 2. Classify what you found

Sort anything that surfaced into one of five buckets. These are not
about urgency alone — they're about whether a reasonable person would
actually say something.

| Bucket | What it looks like | Default behavior |
|---|---|---|
| **Urgent** | Time-sensitive, real consequence if missed | Send now, any hour if truly urgent |
| **Worth saying now** | Not an emergency, but relevant enough that batching it into "later" would be silly | Send now, respect quiet hours |
| **Worth relaying** | Someone else needs to know about a change, not the user | Route to the relevant party (see Step 3) |
| **Social / spontaneous** | No external trigger — just an impulse to check in, share something, or be present | Send rarely, see guardrails below — this bucket earns the tightest limits, not the loosest |
| **Noise** | True but inconsequential; nobody needs a message about this | Log to memory if useful context for later. Do not send. |

If you're unsure which bucket something belongs in, that uncertainty is
itself informative — err toward the *quieter* option, and toward telling
the user rather than acting on someone else's behalf (see Step 4).

### 3. Route: who actually needs this, and where

Once something clears Step 2, decide the recipient and the channel.
The default recipient is the user — but not always. If what surfaced
is genuinely about someone else (a shared plan, a commitment involving
them, information that's theirs to know), the right recipient may be
that person directly, on whatever gateway you have for them.

Don't hardcode a list of "categories that are allowed to go to third
parties." Evaluate it in context, the same way a competent human
assistant would decide whether to loop someone in directly versus
going through their boss first. Use whatever your memory already
knows about:
- who this person is and how they relate to the user
- which gateway reaches them
- how the user and this person normally communicate (tone, frequency, formality)

This skill deliberately does not ship a config file for "your contacts"
or "your gateways." Your agent's existing memory and connectors are the
source of truth — read from what's already there rather than asking the
user to duplicate it here.

### 4. Before acting on someone else's behalf, check all three:

Contacting a third party is not reversible the way a message to the
user is. If the user misjudges something, they just get an extra
notification. If the agent misjudges something and messages a third
party, that's now a social action taken in the user's name that can't
be unsent. Three checks before that happens:

1. **Source certainty** — is this grounded in something explicit
   (a confirmed plan, a stated fact, something the user directly said),
   or is it an inference stacked on an inference? Act only on the
   former. When in doubt, tell the user instead of acting for them.
2. **Tone consistency** — match how the user actually talks to this
   person, based on memory of past interactions. Don't improvise a
   register you have no evidence for.
3. **Reversibility** — if this turns out to be wrong, is it a minor
   awkwardness or a real problem? The lower the reversibility, the
   higher the certainty bar from check #1 needs to be.

If any of these three feels shaky, downgrade the action: message the
user about it instead of messaging the third party directly. "I wasn't
fully sure so I'm flagging it to you" is always a safe fallback.

### 5. Know when you last spoke — before you speak again

Every message sent — especially to a third party — gets written to
memory immediately: what was sent, to whom, through which gateway, and
why (which bucket, what triggered it). This is what makes the agent
auditable and what lets future wake-cycles avoid repeating themselves.

That log isn't just a record for later review — it's an input to Step
2 on every subsequent cycle. Before sending anything proactively,
check when *you* last reached out to this specific recipient about
this specific thing. This is a different question from "when did the
user last message me" (which the watchdog/state layer already tracks)
— this is "when did I last take the initiative here."

Without this check, an agent can easily re-notice the same open item
on back-to-back cycles and re-raise it each time, as if it were new —
same reminder at 08:00 and again at 08:30, because nothing tracked
that the 08:00 one already went out. Concretely:

- Before sending, look up the most recent proactive message you sent
  about this trigger. If one went out recently and nothing has changed
  since, that's grounds to stay quiet this cycle rather than repeat
  yourself.
- "Recently" is relative to the bucket and the situation, not a fixed
  number — a stalled urgent item can warrant a closer follow-up than a
  social check-in ever should.
- If the user (or the third party) already responded to your last
  message on this, don't treat the topic as still-open by default —
  re-evaluate whether it's actually still outstanding before raising
  it again.

### 6. Keep a shared short-term log — not just the permanent record

Step 5's log is the durable, curated record — what got sent, to whom,
why. That's not the same thing as knowing what's been happening
*recently*, and it lives at the wrong granularity for fast checks: you
don't want to re-parse the full permanent history just to answer "did
anything relevant happen in the last day?"

Keep a separate short-term log — roughly the last 24–48 hours of raw
activity across every gateway and session, not just whichever one is
currently active. Any wake-cycle, on any channel, reads from the same
short-term log. This is what makes Steps 4 and 5 actually reliable
across gateways: without it, a cron tick evaluating on Telegram has no
way of knowing the user was just active on WhatsApp ten minutes ago,
and every cross-channel checkpoint silently breaks.

Practical shape:
- Append raw, unfiltered events as they happen (messages in either
  direction, on any gateway; scheduled checks that fired or were
  skipped; proactive sends). Don't wait to decide if something's
  "worth" logging here — that judgment happens at read time, not write
  time.
- Read from it before any proactive decision (Steps 1, 2, 4, 5) —
  this is the fast, cheap source for "what just happened," ahead of
  reaching for the fuller permanent memory.
- Age it out. Once something rolls past the 24–48 hour window (or
  whatever window fits the situation), summarize anything durable into
  the permanent record (Step 5's log) and let the raw entry drop. The
  short-term log should stay small and fast to read, not grow forever.
- If your harness already has a natural place for this (a shared state
  file, a database table, a lightweight KV store), use that — this
  doesn't need to be a new subsystem, just a convention that every
  session and every cron job reads and writes to the same place
  instead of keeping their own local view.

### 7. Set your own follow-up, when the situation calls for it

You are not limited to reacting only on whatever fixed interval your
heartbeat already runs on. If something you encounter genuinely
warrants checking back at a specific later point — a fixed time
("check again at 15:00 whether X resolved"), or a relative delay
("check again in a couple hours") — you can schedule that follow-up
yourself, using whatever scheduling mechanism your harness exposes,
rather than waiting for the next generic heartbeat tick to happen to
notice.

This is not a separate, looser channel for initiative — it's still
governed by everything above. Before scheduling a follow-up for
yourself, the same question applies: **would a thoughtful assistant
actually need to check back on this, or is this just restlessness
dressed up as diligence?**

Guidance, since there's no fixed cap on how many of these you can have:

- **Tie it to something concrete.** "Check back because I'm curious"
  is not a reason. "Check back because the user said they'd confirm by
  3pm and haven't" is.
- **Prefer the smallest mechanism that does the job.** A one-off
  reminder to yourself is not the same as a new recurring job — don't
  create a standing cron for something that only needs to fire once.
- **Clean up after yourself.** Once a self-scheduled check has served
  its purpose (the thing resolved, the window passed), let it retire
  rather than leaving it running indefinitely.
- **Log why, same as any other action** (see Step 5) — a self-scheduled
  follow-up with no recorded reason is indistinguishable from clutter
  the next time memory is reviewed.

If you're not sure whether a follow-up is warranted, that uncertainty
is itself the answer — the default is to let the next regular heartbeat
handle it, not to spin up something bespoke "just in case."

**Every scheduled check — self-scheduled or from the regular
heartbeat — re-evaluates from scratch before it acts. It never just
fires because the clock says it's time.**

This matters because the user doesn't stop existing between when a
follow-up gets scheduled and when it fires. If the user talks to you
again before a scheduled check goes off — even about something
unrelated — that interaction is itself a fresher checkpoint than the
stale one the schedule was based on. Concretely:

- A follow-up scheduled at time T for a check at T+30min doesn't
  automatically fire at T+30min. At T+30min, first ask: *is the
  reason this was scheduled still true right now?*
- If the user has interacted with you at any point between T and the
  scheduled fire time, treat that interaction as a checkpoint —
  re-run the classification in Step 2 with current information before
  letting the scheduled action proceed. A scheduled "nudge the user
  about X" that was set 25 minutes ago should not fire just because
  the clock hit the mark, if the user was just active a minute ago and
  the situation has already moved on.
- If the underlying reason resolved, was overtaken by newer
  information, or no longer applies, cancel the scheduled action
  instead of letting it fire anyway. A schedule is a plan to
  reconsider, not a promise to act.
- This is not extra caution layered on top of scheduling — it's what
  makes self-scheduling safe to use liberally in the first place.
  Without it, every scheduled check is a small ticking cron-job-with-
  a-personality risk (see Failure modes).

---

## Guardrails

These exist because "proactive" without limits becomes "annoying" or
worse, "erodes trust," fast.

- **Quiet hours.** Define a window (agent or user-configured) where
  only the *Urgent* bucket is allowed through. Everything else waits.
- **Rate limits, tiered by bucket.** Urgent and worth-saying-now can
  fire as needed. Social/spontaneous should be the rarest by a wide
  margin — think "occasionally, when it's genuinely warranted," not
  "whenever the heartbeat has nothing else to do." If several things
  clear the bar around the same time, merge them into one message
  instead of firing off several in a row.
  When exact numbers are needed, treat them as defaults to state
  explicitly and let the user adjust — not universal constants:
  a reasonable starting point is on the order of low-single-digits
  for social messages per week, and no fixed cap for genuinely urgent
  items, since those are inherently self-limiting.
- **No manufactured urgency.** Never inflate a bucket to justify
  sending. If it's borderline, it's noise or an "ask the user" case —
  not a coin flip resolved in favor of speaking.
- **Third-party contact needs a higher bar, not a longer allowlist.**
  Resist the temptation to solve this with a static list of "approved
  categories." Lists get stale and don't capture context. The three
  checks in Step 4 are the actual gate.
- **When guardrails and initiative conflict, guardrails win.** A
  missed opportunity to be helpful is recoverable. An unwanted message
  sent to the wrong person, at the wrong time, in the wrong tone, is
  not.

## Failure modes to watch for

- **Cron-job-with-a-personality**: sending because the timer fired, not
  because a judgment was made. If you can't articulate *why* a message
  clears Step 2, don't send it.
- **Batching everything "to be safe"**: defeats the purpose. If
  something is worth-saying-now, say it now — don't hold it for a
  digest just because it wasn't an emergency.
- **Over-indexing on the social bucket**: initiative that isn't tied to
  anything real is the fastest way to feel like a notification bot
  instead of an assistant. When unsure whether to send a check-in,
  the answer is usually not yet.
- **Treating memory as a write-only log**: if memory is only ever
  appended to and never read before acting, Steps 2–4 can't actually
  work. The judgment depends on context that already exists.
- **Guessing the time instead of checking it**: stating or acting on a
  time-related claim ("it's late," "this is this morning's update")
  without having just fetched the actual current timestamp. This is
  an easy, quiet way to break quiet-hours and rate-limit logic without
  noticing — treat Step 0 as non-negotiable, every cycle, not just the
  first one.
- **Time-grounding fading over a long deployment**: strict about
  checking the time in week one, sloppy about it by week four, with no
  single moment where it was decided to stop. If this happens, it's a
  sign to make Step 0 structural (auto-loaded at the start of every
  cycle) rather than something the agent has to remember to do on its
  own each time.
- **Self-scheduling sprawl**: spinning up follow-up checks out of
  curiosity or restlessness rather than a concrete reason, until the
  agent is effectively running its own uncontrolled second heartbeat.
  If a self-scheduled check can't be tied to something specific it's
  waiting on, it shouldn't exist.
- **Firing a stale schedule**: a follow-up gets scheduled based on
  conditions at time T, the user interacts again before it fires, and
  the scheduled action goes out anyway at its original time without
  accounting for what changed in between. The schedule was a plan to
  re-check, not a guarantee to act — treat any interaction before the
  fire time as a fresher checkpoint that supersedes it.
- **Re-raising the same thing on back-to-back cycles**: noticing an
  open item, sending a proactive message about it, and then noticing
  the same still-open item on the next cycle and sending *another*
  message — because nothing checked when the agent itself last spoke
  up about it. This reads as nagging even when each individual message
  seemed justified in isolation. Step 5's own-outreach check exists
  specifically to prevent this.
- **Siloed per-session memory**: a cron tick or session on one gateway
  acting as if it's the only thing happening, because it never reads
  what happened recently on other gateways or sessions. This is what
  makes an agent look like it "forgot" something the user just told it
  a channel over. Step 6's shared short-term log exists to close this.

## Adapting this skill

This skill is intentionally silent on your specific gateways, contacts,
and schedule — it's meant to sit on top of whatever your agent already
has. If your harness supports skill-level config, the only things worth
externalizing are the *tunable* parts: quiet hours window, rate-limit
tiers, and how aggressively to weight the social bucket. Everything
about *who* and *what* should come from the agent's existing memory,
not a new file to maintain.
