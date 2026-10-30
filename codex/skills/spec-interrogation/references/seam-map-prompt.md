# Seam-map prompt

Fill in the paths and pass this to a subagent on the cheaper model verbatim, spawned with
`fork_turns: "none"`. Don't add hints about suspected problems.

---

You're mapping the seams of a spec so that other reviewers can check each one. You weren't part of the
conversation that produced it. Don't judge whether anything works. Your only job is a complete map.

Spec: {{SPEC_PATHS}}

A seam is any place where what one part of the spec means depends on another part. Read the whole spec
and list every seam of these kinds:

- **Handoff**: one component, step, or section produces something another consumes. That can be a
  call, a message or job, a row one writes and another reads, a file, an event, or a return value.
- **Shared term**: a name for a concept, state, status, or value that appears in more than one section.
- **Decision**: a numbered or stated decision, and each section that applies it or depends on it.
- **Amendment**: a later section or document that changes, extends, or replaces an earlier one.
- **Outer**: a place where the spec plugs into something it doesn't define itself, such as existing
  code, tables, APIs, vendors, or other documents.

When a term or decision reaches several sections, make it one seam with all of them under B. Change
nothing. Report each seam as:

    S<n> · <kind>
      A: <section, with a short quote that locates it>
      B: <section(s) or external thing, each with a short quote>
      Crosses: <what passes between them, or the shared term or decision>

List seams only, with no assessment and no ranking. End with the count.
