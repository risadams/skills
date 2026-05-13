---
name: clarity-council
description: Facilitates structured single-persona or multi-persona consultation for strategy, product, engineering, operations, and risk decisions. Use when the user requests a council, multiple perspectives, persona advice, tradeoff analysis, or iterative clarification and debate.
---

# Clarity Council

Use this skill to run persona-based consultation with consistent structure and explicit tradeoffs.

## When to use

Use this skill when the user asks for any of the following:
- multiple expert viewpoints
- one specific persona's recommendation
- synthesis of agreements, conflicts, risks, and next steps
- iterative back-and-forth to clarify requirements before final guidance

Do not use this skill for pure factual lookup with no decision component.

## Inputs

Collect what is available, then proceed:
- required: `user_problem` or `requestText`
- optional: `context`
- optional: `desired_outcome`
- optional: `constraints` (list)
- optional: `selected_personas` or `personasRequested` (list)
- optional: `depth` (`brief` | `standard` | `deep`)
- optional for iterative sessions: `state`, `answer`, `sessionId`

## Workflow

Use this checklist for complex requests:

```text
Council Progress:
- [ ] Step 1: Classify mode (single persona, multi-persona, or multi-turn discussion)
- [ ] Step 2: Load persona contracts and apply overrides if requested
- [ ] Step 3: Produce persona responses with explicit assumptions
- [ ] Step 4: Synthesize agreements, conflicts, risks/tradeoffs, next steps
- [ ] Step 5: If ambiguity remains, ask targeted clarification and continue
```

### Step 1: Classify mode

- Single persona request: use [skills/persona_consult.md](skills/persona_consult.md)
- Multi-persona synthesis request: use [skills/council_consult.md](skills/council_consult.md)
- Iterative discussion request: use [skills/council_discuss.md](skills/council_discuss.md)

### Step 2: Load persona contracts

- Read [skills/personas/PERSONAS.md](skills/personas/PERSONAS.md)
- If overrides are requested, follow [skills/council_define_personas.md](skills/council_define_personas.md)

### Step 3: Generate persona responses

Follow [skills/persona_consult.md](skills/persona_consult.md) for each persona.

### Step 4: Synthesize

Follow [skills/council_consult.md](skills/council_consult.md) and produce:
- agreements
- conflicts
- risks_tradeoffs
- next_steps

### Step 5: Clarify and iterate when needed

For unresolved ambiguity or competing constraints, use [skills/council_discuss.md](skills/council_discuss.md).
Return updated session state each turn.

## Output templates

Use these structures exactly unless the user asks for a different format.

### Single persona output

```markdown
summary: <short answer>
advice:
- <point>
assumptions:
- <assumption>
questions:
- <question>
next_steps:
- <action>
confidence: <low|medium|high>
```

### Multi-persona output

```markdown
responses:
- persona: <name>
  summary: <short answer>
  advice:
  - <point>
  assumptions:
  - <assumption>
  questions:
  - <question>
  next_steps:
  - <action>
  confidence: <low|medium|high>

synthesis:
  agreements:
  - <point>
  conflicts:
  - <point>
  risks_tradeoffs:
  - <point>
  next_steps:
  - <action>
```

### Multi-turn discussion output

```markdown
status: <needs_clarification|in_progress|completed>
message: <short user-facing response>
nextAction: <what is needed next>
output:
  <single-persona or multi-persona structure>
state:
  sessionId: <id>
  turn: <number>
  history:
  - <compact turn summary>
```

## Reference files

Read these files directly from this list when needed:
- [EXAMPLES.md](EXAMPLES.md) — input-only invocation examples for all modes
- [skills/persona_consult.md](skills/persona_consult.md)
- [skills/council_consult.md](skills/council_consult.md)
- [skills/council_define_personas.md](skills/council_define_personas.md)
- [skills/council_discuss.md](skills/council_discuss.md)
- [skills/personas/PERSONAS.md](skills/personas/PERSONAS.md)
- [skills/SKILL_GRAPH.md](skills/SKILL_GRAPH.md)
- [skills/RUNBOOK.md](skills/RUNBOOK.md)

## Quality checks

Before finalizing a response:
- ensure recommendations tie to stated constraints
- separate assumptions from facts
- include at least one concrete next step
- include at least one explicit risk or tradeoff for non-trivial decisions
- keep terminology consistent with persona names in [skills/personas/PERSONAS.md](skills/personas/PERSONAS.md)
