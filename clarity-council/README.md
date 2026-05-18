# Clarity Council

Run a structured persona-based consultation for any decision that benefits from multiple perspectives — strategy, product, engineering, operations, risk. The skill supports three modes (single persona, multi-persona synthesis, iterative discussion), enforces consistent output structure (assumptions, advice, conflicts, tradeoffs, next steps), and is invoked directly by several other skills (issue-triage, issue-feature-breakdown, issue-estimate-sp, codebase-improve-architecture, mr-draft).

## Why this exists

Most decisions die in one of two ways: a single perspective dominates and the blind spots ship into production, or you collect five opinions in a meeting and walk out without a synthesis. Persona-based consultation catches both: each persona is constrained to a defined viewpoint (so blind spots can't dominate), and the synthesis step is non-optional (so disagreements get articulated, not buried). The skill also makes the consultation portable — the output structure is consistent across modes and across invoking skills, so you can reason about the same decision the same way no matter which entry point you used.

## Triggers

Invoke this skill by:

- Saying any of these in chat:
  - "convene a council"
  - "give me multiple perspectives on this"
  - "what would a {persona} say about X"
  - "stress-test this decision with a council"
- Running the slash command: `/clarity-council`
- Other skills invoke this automatically: `issue-triage`, `issue-feature-breakdown`, `issue-estimate-sp`, `codebase-improve-architecture`, `mr-draft` all delegate to it for their analysis phases.

## What it does

The skill classifies the request into one of three modes, loads persona contracts (with optional overrides), generates persona responses with explicit assumptions, and synthesizes agreements/conflicts/risks/next steps. Iterative mode supports multi-turn discussions with session state. Output is structured (YAML-style markdown blocks) so downstream consumers (other skills, dashboards) can parse it. It does **not** make the final decision — the user does — and it doesn't pretend confidence it doesn't have (every response includes a confidence level).

### Inputs

- **`user_problem`** — required. The decision or question.
- **`context`** — optional. Background, constraints, prior decisions.
- **`desired_outcome`** — optional. What format/depth the user wants back.
- **`constraints`** — optional list. Hard requirements the response must respect.
- **`selected_personas`** — optional list. Default panels are mode-specific.
- **`selected_persona_groups`** — optional list. Use pre-made small panels instead of hand-picking every persona.
- **`depth`** — optional: `brief` / `standard` / `deep`.
- **`state`, `answer`, `sessionId`** — optional. For iterative discussion mode.

### Outputs

- One of three structured markdown blocks (single-persona, multi-persona, or multi-turn discussion). Each contains `assumptions`, `advice`, `questions`, `next_steps`, and a `confidence` value.

### External systems used

- Local filesystem (reads persona contracts from `skills/personas/`)

## How to use it

A typical multi-persona session looks like this:

```text
You: convene a council on whether to migrate our auth from basic to OAuth.
     Personas: senior-architect, security-engineer, product-owner.

Skill: [Loads persona contracts, generates responses]

       responses:
       - persona: senior-architect
         summary: Migrate; the operational complexity of basic auth at our
           scale already exceeds OAuth's setup cost.
         advice:
         - Use a library, not a hand-rolled implementation.
         - Plan a 4-6 week parallel-run window before cutover.
         assumptions:
         - The team has at least one engineer with prior OAuth experience.
         confidence: high

       - persona: security-engineer
         summary: Migrate, but only after threat-modeling the new attack surface.
         advice:
         - JWT in httpOnly cookies, not localStorage.
         - Token refresh must be idempotent and observable.
         confidence: high

       - persona: product-owner
         summary: Don't migrate this quarter — Q3 has 3 in-flight payment
           features and migration risk would compete for the same eng cycles.
         confidence: medium

       synthesis:
         agreements:
         - Migration is the right direction.
         conflicts:
         - When: senior-architect + security-engineer want it now;
           product-owner wants Q4.
         risks_tradeoffs:
         - Q3 conflict with payments work is the load-bearing tradeoff.
         next_steps:
         - Decide cutover quarter (Q3 vs Q4) before scoping further.
         - Threat-model the new attack surface regardless of timing.
```

## Getting the most out of it

- **Pick personas deliberately.** Default panels work, but `selected_personas` lets you assemble the right room. A council without `devils-advocate` will drift toward consensus prematurely.
- **Always use the synthesis.** The synthesis is what makes multi-persona output usable. Skipping straight to advice from your favorite persona defeats the point.
- **Use `depth: brief` for routine decisions.** A single-persona quick consult takes seconds; the full multi-persona deep mode takes much longer. Match the depth to the stakes.
- **Read agreements before conflicts.** Agreements are the strong evidence; conflicts are the live questions. Many users skim straight to conflicts and miss what every persona already concluded.

## Anti-patterns

What this skill will NOT do, or what to avoid:

- ❌ **Make the final decision.** The synthesis surfaces tradeoffs and next steps; the human decides. The skill won't say "do X" with no qualification.
- ❌ **Use it for pure factual lookup.** "What's the syntax for a Postgres index?" doesn't need a council. Save the skill for decisions, not facts.
- ❌ **Run a council in a hurry without context.** Garbage in, structured-garbage out. If the problem statement is two sentences and there's no context, the personas have nothing to push on.
- ❌ **Skip the assumptions list.** Every response includes assumptions explicitly. Ignoring them turns the advice into "trust me" — and the assumptions are usually where the real disagreement lives.

## Examples

### Example: Single-persona quick consult

```text
You: as a senior-developer: should we cache the user-profile lookup
     or fetch it on every request? The profile is ~2KB and changes
     a few times per session.

Skill: summary: Cache it with a short TTL — 30s.
       advice:
       - In-memory cache (process-local) is enough; Redis is overkill.
       - Stamp every cache entry with a `last_modified` from the source.
       assumptions:
       - User load is modest (< 1k req/s/instance).
       - Profile mutations are infrequent enough that 30s staleness
         is acceptable to the product.
       questions:
       - Is there an SLA on profile-update visibility?
       next_steps:
       - Confirm the SLA, then ship the cache.
       confidence: high
```

Single-persona is fast and gets you a recommendation with the assumptions exposed.

### Example: Iterative discussion (multi-turn)

```text
You: convene a council on adopting feature flags. I'll iterate.

Skill: status: needs_clarification
       message: Is the goal experimentation, gradual rollout, or kill-switching?
       Each implies different infrastructure.

You: gradual rollout primarily; experimentation maybe later.

Skill: status: in_progress
       output:
         (multi-persona output for "gradual rollout feature flags")
       message: I have a recommendation. Want me to deepen any conflict?
```

Multi-turn mode keeps state across turns, refining the recommendation as constraints emerge.

## Internals

The skill follows a 5-step workflow per request:

1. **Classify mode** — single persona, multi-persona synthesis, or iterative discussion.
2. **Load persona contracts** — from `skills/personas/PERSONAS.md`; apply overrides if `council_define_personas` was invoked.
3. **Generate persona responses** — each persona produces summary / advice / assumptions / questions / next_steps / confidence following its contract.
4. **Synthesize** — agreements / conflicts / risks_tradeoffs / next_steps across personas (multi-persona modes only).
5. **Clarify and iterate** — for unresolved ambiguity in iterative mode, return state and ask targeted questions.

Three modes:

- **`persona_consult`** — single persona, structured output.
- **`council_consult`** — multi-persona with synthesis.
- **`council_discuss`** — multi-turn iterative discussion with session state.

Persona contracts (`skills/personas/PERSONAS.md`) define each persona's frame: senior-architect (structural causes), senior-developer (implementation-level causes), qa-engineer (test/data/environment), security-engineer (attack surface), product-owner (scope/value/timing), scrum-master (velocity/capacity), tech-lead (technical complexity/dependencies), devils-advocate (challenges the obvious).

Quality checks every response:

- Recommendations tie to stated constraints.
- Assumptions are separated from facts.
- At least one concrete next step.
- At least one explicit risk or tradeoff for non-trivial decisions.
- Persona terminology is consistent with the contracts.

## FAQ

**Q: What's the default panel for `council_consult`?**
A: It depends on the invoking skill (e.g., issue-triage uses senior-architect + senior-developer + qa-engineer + devils-advocate). For direct invocation, the skill picks based on the problem type or asks.

**Q: Can I add custom personas?**
A: Yes — invoke `council_define_personas` with the persona spec. The override applies for the session.

**Q: How is this different from grill-me?**
A: Grill-me interrogates *you* one question at a time to surface gaps in your plan. Clarity-council collects multiple opinions on a decision and synthesizes. Different shapes; different inputs.

**Q: Is the iterative mode worth the overhead?**
A: For meaty multi-faceted decisions (architecture, hiring, vendor selection), yes. For one-shot questions, single-persona or council_consult is faster.

**Q: Why YAML-style markdown for the output?**
A: Other skills consume the output programmatically. Consistent structure means a downstream skill can extract the synthesis section without parsing free-form prose.

## Related skills

- **[issue-triage](../issue-triage/)** — Phase 5 invokes this skill with senior-architect + senior-developer + qa-engineer + devils-advocate.
- **[issue-feature-breakdown](../issue-feature-breakdown/)** — Phase 2 invokes this with senior-architect + product-owner + tech-lead + qa-engineer + devils-advocate.
- **[issue-estimate-sp](../issue-estimate-sp/)** — Step 4 invokes this with scrum-master + tech-lead + senior-developer + qa-engineer for scrum-poker estimation.
- **[grill-me](../grill-me/)** — for stress-testing your own plan one question at a time, instead of getting other voices.
- **[grill-with-docs](../grill-with-docs/)** — like grill-me but anchored against project documentation.

## Files

- **[SKILL.md](SKILL.md)** — Skill entry point (mode classification, output templates, quality checks)
- **[EXAMPLES.md](EXAMPLES.md)** — Input-only invocation examples for all three modes
- **[skills/](skills/)** — Mode implementations (`persona_consult.md`, `council_consult.md`, `council_discuss.md`, `council_define_personas.md`) and persona contracts (`personas/PERSONAS.md`)
