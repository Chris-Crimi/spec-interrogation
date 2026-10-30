# Audit lenses

Each lens is a question to ask of every section. The examples are there to show the shape of each
failure. They aren't a checklist of topics.

## Loose threads: introduced, never finished

1. **Named but unowned.** A concept, table, component, or role appears with no statement of who creates
   it, who changes it, and what ends it. Follow every noun to its full lifecycle: created → changed →
   read → retired or deleted → what happens on failure.
2. **States with no way out.** A status that can be entered but never left, or left only by an action the
   spec never describes. The same goes for timeouts, retries, and dead letters: what happens to the thing
   that got stuck?
3. **"We'll handle X."** Promises, TODOs, "later", "as needed", "etc.", "and so on". Each one is either
   resolved, cut, or parked with a trigger.
4. **Deferrals with hidden carrying costs.** Doing it later is cheap only if the data it needs is
   already recorded. Ask what would have to be migrated, backfilled, or broken to add it. If the answer
   isn't "nothing", something has to be in the spec now.
5. **Load-bearing unknowns.** An open question (vendor terms, capability, a pending ruling) that several
   decisions quietly depend on. Name what it blocks and when it has to be answered.
6. **Silent defaults.** Anything an implementer must decide that the spec doesn't mention: notifications,
   error display, concurrency, pagination, limits, retention, who sees what. If it's unstated, the first
   feature that needs it will invent it.
7. **Stale context.** Recommendations written for a situation that has since changed (existing code
   versus greenfield, a different host, a vendor ruled out). Do they still apply, and does anything
   depend on them?

## False joins: look compatible, aren't

1. **One word, two meanings.** The same term used for different things in different sections, or two
   state machines that share a value name by coincidence. Either rename one or add a glossary line saying
   they're unrelated.
2. **Two homes for one fact.** Two mechanisms that record or decide the same thing (a "reviewed" flag
   *and* open tasks, for example). They'll drift apart. Keep one and derive the other.
3. **Layers treated as substitutes.** Two controls that sound redundant but answer different questions
   (one decides whether a request may proceed, the other limits what the code can touch if that
   decision was wrong). Then check for **paths that bypass a layer entirely**: background jobs, queue
   consumers, scheduled tasks, admin scripts, and webhooks never pass through request middleware.
4. **Discipline presented as a guarantee.** A protection that holds only if everyone remembers (a
   convention, a code-review rule, a key that's available but "shouldn't be used"). Ask what fails, and
   how loudly, when someone forgets. Structural alternatives are a missing permission, a constraint, a
   single chokepoint module, and a CI check that breaks the build.
5. **Handoff mismatches.** At every boundary between components: does the producer provide what the
   consumer assumes? Check identifiers (which id, which format), ordering, timing (sync versus async,
   before versus after commit), units, transaction scope, idempotency, and **who handles the failure
   case**. Two sides that each assume the other one handles it is the classic false join.
6. **Chain properties.** A property that has to hold end to end (compliance, isolation, encryption,
   audit completeness, data residency) holds only if *every* hop on the data path has it. Walk the path
   and list each hop, especially the third-party vendors.
7. **Overclaims.** A statement that's true in a weaker form than it's written. Separate the flatly true
   part from the part that needs a qualifier.
8. **Section contradictions.** A later section or amendment changed a decision and an earlier section
   still states the old one. Also look for numbers, limits, or names that differ between sections.

## Assumptions

For each significant decision, ask:

- What premise does this rest on, and was it verified or assumed?
- What's the strongest argument for the alternative that was rejected?
- If the premise is wrong, what breaks, and how would we find out?
- Is this decision reversible? If not, is it made with the confidence that deserves?
