# HEARTBEAT.md (example)

This is a sample checklist a heartbeat/cron mechanism can hand to the
agent on each wake cycle. Adapt the categories to whatever you actually
track — this is a starting skeleton, not a fixed schema.

On each wake, work through this list. If nothing below applies, exit
immediately with no message sent (see `SKILL.md` for the exit
convention your harness uses, e.g. a short `HEARTBEAT_OK`-style
sentinel instead of a full response).

## 1. Standing reminders / deadlines
- Anything in memory with a due date within the next relevant window?
- Anything the user asked to be reminded about that hasn't fired yet?

## 2. Stalled follow-ups
- Any conversation or task where the agent (or a third party) was
  expecting a reply that hasn't come, past a reasonable window?
- Any commitment the user made that's approaching and hasn't been
  confirmed?

## 3. State changes in tracked projects/areas
- Has anything the agent is tracking (a project, a repo, a ticket,
  an inbox) changed since the last check in a way that's actually
  relevant — not just "different"?

## 4. Third-party-relevant items
- Is there anything surfaced above that's actually about someone
  other than the user — something they'd want to know directly,
  not secondhand through the user?
- If yes, this goes through the three-check gate in `SKILL.md` §4
  before any message is sent to them.

## 5. Social / no-trigger check
- Has it genuinely been long enough, and is there a real, specific
  reason to reach out with no other trigger present?
- This bucket has the tightest rate limit of all five (see
  `SKILL.md` guardrails). When in doubt here, the answer is "not yet."

---

**Exit conditions:**
- Nothing above applies → exit silently, no message.
- Something applies → classify it (`SKILL.md` §2), route it (§3),
  and if it involves a third party, run the three-check gate (§4)
  before sending.
- Every message sent gets logged per `examples/memory-log-format.md`
  before the cycle ends.
