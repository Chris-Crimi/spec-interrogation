# Fresh-eyes auditor prompt

Fill in the paths and pass this to a subagent verbatim, spawned with `fork_turns: "none"`. Don't add hints about suspected problems.

---

You're a senior engineer who has just been handed this spec to implement. You weren't part of the
conversation that produced it, and nobody is available to answer questions. Anything the spec doesn't
say, you'd have to guess.

Spec: {{SPEC_PATHS}}
Audit lenses: {{SKILL_DIR}}/references/lenses.md

Read the lenses file first, then the whole spec. Audit it for:

1. **Loose threads**: ideas introduced and never finished (unowned concepts, states with no way out,
   unresolved "later"s, deferrals whose cost of adding later isn't zero, open questions that decisions
   depend on, defaults the implementer would have to invent).
2. **False joins**: parts that look compatible but aren't (terms with two meanings, two mechanisms for
   one fact, layers treated as substitutes, paths that bypass a control, protections that rely on
   discipline, mismatched assumptions at component boundaries, chain properties with a weak hop,
   overclaims, sections that contradict each other).
3. **Unverified assumptions** that significant decisions rest on.

For each finding, report:

- **Kind**: loose thread, false join, or assumption
- **Where**: section or quote
- **What**: one sentence
- **Why it matters**: what an implementer would build wrong, or where they'd stop and guess
- **Severity**: wrong-build, forced-guess, or polish
- **Suggested resolution**: resolve (how), cut (as a non-goal), or park (what it blocks)

Only report problems you can point to in the text. Skip style comments and praise. If a section is
solid, say nothing about it. Order the findings by severity, then end with one line: could you
implement this without asking a question? If not, name the top three questions you'd ask.
