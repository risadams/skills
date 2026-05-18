# Skill: council_discuss

## Purpose

Run a multi-turn council session with clarifications and evolving recommendations.

## Input

- `requestText` (required)
- `state` (optional)
  - `sessionId`
  - `turn`
  - `history[]`
- `answer` (optional)
- `personasRequested` (optional list)
- `personaGroupsRequested` (optional list)

## Output

- `status`
- `message`
- `nextAction`
- `output` (council synthesis for this turn)
- `state` (returned, for explicit handoff)

## Process

1. If no `state`, initialize session with a new `sessionId` and `turn: 1`.
2. Ask clarification when ambiguity is high. Limit to one focused question per turn.
3. For each completed turn, run [council_consult.md](council_consult.md). Expand `personaGroupsRequested` through [council_define_persona_groups.md](council_define_persona_groups.md) before calling the consult step.
4. Apply conflict protocol when personas disagree:
   - Name the dimension of conflict (see council_consult.md synthesis dimensions)
   - Present two or three explicit decision options with a tradeoff table
   - Ask one clarifying question to resolve the conflict before proceeding
5. Return updated `state` so any harness can continue the session.

## Conflict Protocol

When two or more personas hold irreconcilable positions:

1. Label the conflict type: cost, risk, reversibility, speed, people impact, or confidence
2. Surface the conflicting options as a table:
   - Option | Benefit | Risk | Who it favors
3. Ask the user one targeted question to break the tie
4. Record the user's decision in `state.history` before continuing

## Calls

- [council_consult.md](council_consult.md)
- [persona_consult.md](persona_consult.md)
- [council_define_persona_groups.md](council_define_persona_groups.md)

## Escalate To

- none
