---
name: spec-interrogation
description: Turn a rough idea into an implementation-ready spec through adversarial conversation, then audit it for loose threads (ideas introduced but never finished) and false joins (parts that look like they fit but don't). Use whenever the user is writing or refining a spec, design doc, plan, PRD, architecture doc, or amendment that coding agents will implement — and especially when they ask to "review for gaps", "check for hanging threads", "what did we forget", "does this hold together", "poke holes in this", "question the assumptions", "check the seams", or "is this ready to hand off". Also use when the user pushes back on a design claim and wants it stress-tested, even if they never say "spec".
---

# Spec interrogation

The user writes specs by talking an idea through with Codex until it holds up, and then hands the spec
to coding agents to build. Those agents build what's written. They can't ask what we meant, and they
fill every gap with a reasonable guess of their own. So a spec is done when nothing in it depends on
context that only lived in the conversation.

Two failure kinds matter most:

- **Loose threads**: something was introduced and never finished. A concept got a name but no owner. A
  "we'll handle that" never got handled. A deferral has no trigger. A question that three decisions quietly
  depend on is still open.
- **False joins**: two parts that look compatible but aren't once you think for a second. A word means
  different things in two sections. Two mechanisms track the same fact. One component assumes the other
  handles the failure case. A layer gets treated as a substitute for another layer that answers a
  different question.

## Two modes

### 1. Interrogate (the default while the idea is taking shape)

Work the idea like a skeptical senior engineer who wants it to succeed:

- **Question the premise before the design.** Ask what problem this solves, what's already true, and what
  changed since the last decision. When the context shifts (greenfield instead of existing code, a new
  constraint, a vendor finding), say which earlier recommendations it invalidates. Don't quietly carry
  them forward.
- **Make claims defensible.** For each strong statement ("X can't do Y", "this is secure", "this
  scales"), separate the part that is flatly true from the part that needs a qualifier, and word it so a
  hostile reviewer can't catch it overclaiming. "Basic header checks work there, deep validation
  doesn't" beats "can't validate files".
- **Steelman the pushback.** When the user relays an outside opinion ("I've heard you could just use X"),
  concede what's right about it first. Then find the specific test that decides the question (walk the
  real workload through the alternative and see where each piece lands) instead of arguing from slogans.
- **Say what each mechanism guarantees.** A protection is either *structural* (the unsafe path doesn't
  exist, or it fails loudly: a constraint, a build failure, a missing grant) or a *discipline* (everyone
  has to remember). Name which one it is. When it's a discipline, propose the cheap move that makes it
  structural, or record it as an accepted risk with conditions for when that ends.
- **Ask one sharp question at a time** when a real decision is needed, with a recommendation. Don't
  interview the user about things you can settle yourself.
- **Write decisions into the spec as they're made**, each with its *why*. A decision that exists only in
  chat is a loose thread by definition.

### 2. Audit (on request, at natural checkpoints, and always before handoff)

The audit has three passes, because they catch different things. Start Passes A and C together (both
are subagents that don't need you), and do Pass B while they run.

**Spawning subagents in Codex.** Use `spawn_agent` with `fork_turns: "none"` for every auditor in
Passes A and C. A fork copies this conversation into the child by default, and an auditor that has seen
the conversation can't catch what the conversation "knew" but the spec never said. Collect the results
with `wait`. If `spawn_agent` isn't available (multi-agent turned off), run each auditor as a separate
process instead, which also starts with no conversation:
`codex exec --ephemeral -s read-only -C <repo> "<filled-in prompt>"`, adding `-m <model>` for Pass C.

**Pass A: fresh eyes (subagent).** Spawn a subagent that sees only the spec file(s), not this
conversation. It reads the way the implementing agent will, so it catches everything we both "know"
but never wrote down. Give it the prompt in `references/auditor-prompt.md`, filled in with the spec
path(s). Don't pass along your own opinions about where the problems are. Priming it defeats the point.

**Pass B: conversation versus spec (you).** Only you have the conversation, so check for drift:

- Things agreed in chat that never made it into the document.
- Edits that landed in one section while the sections that depend on them still say the old thing.
  After any change, grep the spec for the concept's name and check every place it appears.
- Recommendations that were made under a context the user has since changed.
- "Want me to add that?" offers the user said yes to, and whether they actually got written.

**Pass C: seams (a swarm on the cheaper model).** False joins live at the seams: the places where one part hands
something to another, or where two parts lean on the same word, fact, or decision. Each side reads
fine on its own, and the mismatch shows only when both sides are read together, closely. One reader of
the whole spec spreads its attention too thin for that, so each seam gets a narrow agent of its own.

1. **Map the seams.** Spawn one subagent on the fast, affordable coding model (`model: "gpt-reserve"`
   at the time of writing; if that name isn't offered, pick the cheapest coding model listed, or leave
   `model` unset and pass `reasoning_effort: "low"`) with the prompt in
   `references/seam-map-prompt.md`, filled in with the spec path(s). It sees only the spec, lists every
   seam, and judges none of them. If the conversation tells you of a seam the map missed, add it with
   its two sides only. Never say what you suspect about it.
2. **Check every seam.** Spawn one subagent per seam on the same model, all of them before you wait on
   any, so they run at once, each with the prompt in `references/seam-auditor-prompt.md`, filled in with the spec path(s)
   and its seam copied from the map. Keep the swarm to about 15 agents: past that, give one agent up to
   three seams that share a side. Never drop a seam to fit. A spec with fewer than four seams doesn't
   need the swarm, because Pass A covers it.
3. **Treat the results as claims.** Each seam agent saw a slice and runs on the cheaper model, so check
   both of its quoted sides against the spec before you keep a finding. The same mismatch often arrives
   from both ends of a seam and from Pass A, so merge those. In the report, one line says how many seams
   were checked and how many held.

Run Pass C before every handoff and whenever the user asks for a seam check. At other checkpoints,
Passes A and B are enough unless the changes since the last audit touched several sections.

Then **triage** before you report. Verify each subagent finding against the spec: drop what's already
answered, and merge duplicates. For each finding that survives, propose one of three resolutions:

1. **Resolve.** Edit the spec. State the decision and why.
2. **Cut.** Explicitly out of scope. Write it down as a non-goal so nobody builds it on assumption.
3. **Park.** Put it in the spec's Open Threads section with an owner or a trigger, and say what it
   blocks.

Present findings grouped by severity: first what would make an implementer build the wrong thing, then
what makes them stop and guess, then polish. Lead with findings, not a summary of the spec. The lenses
behind each finding kind are in `references/lenses.md`. Read that file before running any pass.

## What the spec must carry for coding agents

Besides the design itself, check that these exist and are current:

- **Decisions with reasons.** Number them so later sections and amendments can refer to them.
- **Non-goals.** Things deliberately not built, stated so no implementer builds them by default
  (for example, "notifications are in-app only; there is no email or push layer").
- **Deferrals, each with a trigger and a carrying cost.** A deferral is cheap only if doing it later
  needs no schema change, backfill, or contract break. Otherwise say what has to exist *now* so it
  stays cheap.
- **Accepted risks**, each with its compensating controls and the conditions that end it.
- **Open Threads.** A standing section that each audit updates: open question → what it blocks →
  owner/trigger. Mark the load-bearing ones, the questions several decisions depend on.
- **Glossary** for any term used in more than one sense nearby, or any name two state machines share
  by coincidence ("these are unrelated; don't couple them").
- **Build order with testable exits**, so an agent knows when a stage is done.

A handoff-readiness test to finish with: could an agent that has never seen this conversation implement
each section without asking a question whose answer we already know? Every "no" is a finding.

## Style

Explain in mechanics and plain language, and be honest about tradeoffs. When the right answer is "it
depends", name what it depends on and give your recommendation. Keep the spec the source of truth: say
what changed and where (section references), not the whole document again.
