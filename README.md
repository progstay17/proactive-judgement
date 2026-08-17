# Proactive Judgment Skill

A portable skill that teaches an AI agent **when to speak first** — not
just how to run on a schedule.

Most "make my agent proactive" setups are really just a cron job with a
personality: a timer fires, a script pulls some data, formats a
message, and pushes it to chat. The agent never actually *decided*
anything. This skill is the missing judgment layer that sits on top of
whatever scheduling/memory/messaging infrastructure your agent already
has, and decides:

- Is this actually worth saying, or is it noise?
- Who should hear it — the user, or someone else entirely?
- Which channel, and in what tone?
- Is this reversible enough to act on autonomously, or should it go
  through the user first?

It does not ship a scheduler, a memory store, or a messaging gateway.
It assumes you already have those (Claude Code + cron, OpenClaw,
Hermes, or a custom harness all qualify) and gives the agent running on
top of them a consistent way to reason about initiative.

## Why this exists

Two observations from looking at how "proactive agent" skills are
usually built:

1. **Proactivity is usually faked.** The agent doesn't wake up and
   reason — a script wakes up, pulls an API, and dumps the result into
   chat. That's automation, not judgment. See ["Your AI Agent Isn't
   Proactive — It's Just a Cron Job With a
   Personality"](https://medium.com/@vivioo.io/your-ai-agent-isn-t-proactive-it-s-just-a-cron-job-with-a-personality-6a42440539e6)
   for the sharpest version of this critique.
2. **Existing proactive-agent skills tend to over-scope.** They bundle
   memory architecture, cron setup, *and* judgment into one package,
   and often assume access to email/calendar/gateway credentials
   without documenting how auth is supposed to work — which makes them
   hard to trust and hard to install safely.

This skill deliberately does one thing: the judgment layer. Bring your
own heartbeat, memory, and gateways.

## Install

Drop the `proactive-judgment/` folder into wherever your harness loads
skills from, e.g.:

```bash
# Claude Code / OpenCode-style project skill
cp -r proactive-judgment/ .claude/skills/

# OpenClaw
openclaw skills install ./proactive-judgment
```

Or point your agent at this repo directly and ask it to install the
skill for itself — most modern harnesses can do this from a URL.

## What's in this repo

| File | Purpose |
|---|---|
| `SKILL.md` | The skill itself — read this first |
| `examples/HEARTBEAT.md` | A sample wake-cycle checklist your heartbeat can hand to the agent |
| `examples/memory-log-format.md` | A sample format for logging proactive actions, so they're auditable later |
| `LICENSE.txt` | MIT-0 — use it, fork it, ship it |

## Design principles

- **General-purpose.** No hardcoded contacts, gateways, or categories.
  The skill reads from whatever memory and connectors your agent
  already has.
- **Silence is a valid outcome — the default, even.** Most wake-cycles
  should end in nothing happening. The skill is built around a fast
  exit path for "nothing worth saying right now."
- **The hardest guardrails are on the most human-feeling behavior.**
  Spontaneous, no-trigger check-ins ("just saying hi") get the
  tightest limits, not the loosest — that's the behavior most likely
  to curdle into spam if left unchecked.
- **Third-party contact is judged by certainty and reversibility, not
  by a static allowlist.** A fixed list of "approved categories" for
  contacting people other than the user goes stale fast and doesn't
  capture context. This skill uses a three-check gate instead (source
  certainty, tone consistency, reversibility) — see `SKILL.md` §4.

## Contributing

Issues and PRs welcome — especially real-world failure cases (agent
sent something it shouldn't have, or stayed silent when it shouldn't
have) that can sharpen the judgment rules in `SKILL.md`.

## License

MIT-0 — see `LICENSE.txt`.
