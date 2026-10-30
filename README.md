# spec-interrogation

A Claude skill for writing specs that coding agents can build without asking questions.

Coding agents build what's written. They can't ask what you meant, and they fill every gap with a
reasonable guess of their own. This skill treats a spec as done when nothing in it depends on context
that only lived in the conversation.

It works in two modes.

**Interrogate**, while the idea is taking shape. Claude acts as a skeptical senior engineer who wants
the idea to succeed. It questions the premise before the design, words each claim so a hostile reviewer
can't catch it overclaiming, steelmans outside pushback, says whether each protection is structural or
relies on discipline, and writes every decision into the spec with its reason.

**Audit**, before handoff or whenever you ask. It hunts for two kinds of failure:

- **Loose threads**: ideas that were introduced and never finished, such as a concept with no owner, a
  "later" with no trigger, or an open question that decisions depend on.
- **False joins**: parts that look like they fit but don't, such as one word with two meanings, two
  mechanisms tracking one fact, or a boundary where each side assumes the other handles failure.

The audit runs three passes:

| Pass | Who | What it catches |
| --- | --- | --- |
| A. Fresh eyes | one subagent that sees only the spec | everything the conversation "knew" but never wrote down |
| B. Drift | Claude itself | things agreed in chat that never reached the document, and edits that landed in one section but not the sections that depend on it |
| C. Seams | a Sonnet swarm: one agent maps every seam, then one agent checks each seam | mismatches that only show when both sides of a boundary are read together, closely |

Every finding is checked against the spec, then resolved (edit the spec), cut (written down as a
non-goal), or parked (an open thread with an owner or a trigger).

## Install

**Claude Code, as a plugin:**

```
/plugin marketplace add Chris-Crimi/spec-interrogation
/plugin install spec-interrogation@spec-interrogation
```

**Claude Code, by hand:** copy `skills/spec-interrogation/` into `~/.claude/skills/` (for all your
projects) or `.claude/skills/` (for one project).

Passes A and C start subagents, so they need a Claude app that can run them, such as Claude Code. In an
app without subagents, interrogation and Pass B still work.

## Use

It triggers on its own when you're writing or refining a spec, design doc, plan, PRD, or architecture
doc, and on phrases like "poke holes in this", "what did we forget", "check the seams", or "is this ready
to hand off". You can also call it by name.

Pass C starts up to about 16 Sonnet agents, so it runs only before handoff, when you ask for a seam
check, or after a change that touched several sections.

## Files

- `skills/spec-interrogation/SKILL.md`: the method.
- `skills/spec-interrogation/references/lenses.md`: the questions behind each kind of finding.
- `skills/spec-interrogation/references/auditor-prompt.md`: the Pass A prompt.
- `skills/spec-interrogation/references/seam-map-prompt.md` and `seam-auditor-prompt.md`: the Pass C
  prompts.

## License

MIT
