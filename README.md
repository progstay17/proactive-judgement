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
- Is it actually the current time and date I think it is — checked,
  not assumed?
- Does this need a follow-up at a specific later point, worth
  scheduling on its own rather than waiting for the next generic
  wake-up?

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

## Reference architecture (optional, one example)

This skill is deliberately silent on which memory/scheduling backend
you use — Step 0's "know the current schedule" and Step 6's "shared
short-term log" are requirements, not a specific implementation. One
real-world way to satisfy both at once: an Obsidian vault as the
agent's long-term memory (e.g.
[obsidian-mind](https://github.com/breferrari/obsidian-mind) or similar
vault-as-memory setups), paired with a local semantic search layer over
that vault (embedding + vector DB, queried before answering, re-indexed
after writes) so the agent can pull relevant context by meaning instead
of exact keywords.

That combination happens to give you both requirements almost for
free: the vault's daily/dated notes double as a natural short-term
log, and a semantic query over it can answer "what's going on right
now" without hardcoding a schedule. But it's one example among many —
a database, a flat file, or your harness's built-in memory works
equally well as long as it satisfies Steps 0 and 6.

## Changelog

- **1.5.0** — Expanded Step 0 from "check the current timestamp" to
  "know the current timestamp and where it sits relative to whatever
  schedule already exists" (upcoming/overdue scheduled items, whether
  now falls inside a defined window). Also named a known failure mode
  directly: time-grounding discipline tends to fade over a long
  deployment even when the instruction hasn't changed, and that's a
  signal to make the check structural rather than memory-dependent.

- **1.4.0** — Added Step 6: a shared short-term log (roughly 24–48
  hours of raw cross-gateway/cross-session activity) distinct from the
  permanent record in Step 5. Generalized from a real multi-gateway
  setup where a per-session view of memory meant a cron check on one
  channel had no idea the user had just been active on another.
- **1.3.0** — Expanded Step 5 (renumbered from a plain "log the
  action" note) to require checking when the agent itself last
  proactively reached out about a given trigger, before sending again
  — not just logging after the fact. Prevents re-raising the same
  open item on consecutive cycles as if it were new.
- **1.2.0** — Added a mandatory freshness check to Step 6 (now Step 7):
  any scheduled follow-up (self-scheduled or regular heartbeat) must
  re-evaluate current conditions before acting, not fire just because
  the clock reached the scheduled time. Generalized from a real race
  condition: a follow-up got scheduled, the user interacted again
  shortly before it was due, and the scheduled action fired anyway
  without accounting for the fresher interaction — a schedule is a
  plan to reconsider, not a promise to act.
- **1.1.0** — Added Step 0 (mandatory current-time grounding before any
  time-related statement or decision) and Step 6 (self-scheduling —
  the agent can set its own follow-up checks, fixed-time or relative,
  when warranted, instead of only reacting on the next fixed heartbeat
  tick). Both were generalized from real-world usage: an agent stating
  a wrong time from a stale assumption, and a need for the agent to
  check back on something at a specific point without waiting on a
  generic interval.
- **1.0.0** — Initial release: five-bucket classification, routing,
  third-party contact gate, guardrails.

## Contributing

Issues and PRs welcome — especially real-world failure cases (agent
sent something it shouldn't have, or stayed silent when it shouldn't
have) that can sharpen the judgment rules in `SKILL.md`.

## License

MIT-0 — see `LICENSE.txt`.
