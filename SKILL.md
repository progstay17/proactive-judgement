---
name: proactive-judgment
description: Give an agent the judgment to decide WHEN to speak first, WHAT deserves initiative, and WHO to route it to — reactive check-ins, follow-ups, or third-party contact. Use when the agent already has a heartbeat/cron mechanism, persistent memory, and at least one outbound message gateway (Telegram, WhatsApp, email, etc.), and the goal is "make my agent proactive" or "make my agent act like a real personal assistant" rather than a task-follower that only replies when spoken to.
version: 1.0.0
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
gateway, and in what tone**.

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

### 5. Log the action, not just the trigger

Every message sent — especially to a third party — gets written to
memory immediately: what was sent, to whom, through which gateway, and
why (which bucket, what triggered it). This is what makes the agent
auditable and what lets future wake-cycles avoid repeating themselves
(don't ask the same question twice, don't re-send a reminder that was
already relayed).

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

## Adapting this skill

This skill is intentionally silent on your specific gateways, contacts,
and schedule — it's meant to sit on top of whatever your agent already
has. If your harness supports skill-level config, the only things worth
externalizing are the *tunable* parts: quiet hours window, rate-limit
tiers, and how aggressively to weight the social bucket. Everything
about *who* and *what* should come from the agent's existing memory,
not a new file to maintain.
