# Skill: council_consult

## Purpose

Consult multiple personas and produce one synthesis.

## Input

- `user_problem` (required)
- `context` (optional)
- `desired_outcome` (optional)
- `constraints` (optional list)
- `selected_personas` (optional list)
- `selected_persona_groups` (optional list)
- `depth` (`brief` | `standard` | `deep`, optional)

## Output

- `responses[]` (per persona)
- `synthesis.agreements`
- `synthesis.conflicts`
- `synthesis.risks_tradeoffs`
- `synthesis.next_steps`

## Process

1. Load active personas from [council_define_personas.md](council_define_personas.md).
2. Load active persona groups from [council_define_persona_groups.md](council_define_persona_groups.md) when the request names a group.
3. Expand `selected_persona_groups` into personas, dedupe against `selected_personas`, and preserve the requested order.
4. For each persona, run [persona_consult.md](persona_consult.md) with full persona contract applied.
5. Synthesize across responses using these five dimensions:
   - **Cost vs Speed**: Which options trade capital for time or vice versa?
   - **Risk vs Reversibility**: Which options are hard to undo and what is the downside?
   - **Short-term vs Long-term**: Which recommendations optimize now at the expense of later?
   - **Confidence**: Which recommendations rest on validated evidence vs assumptions?
   - **People Impact**: Which options create organizational or team burden?
4. In conflicts, name the dimension the personas disagree on — not just that they disagree.
5. Flag if any persona triggered an Escalation Condition.

## Calls

- [persona_consult.md](persona_consult.md)
- [council_define_personas.md](council_define_personas.md)
- [council_define_persona_groups.md](council_define_persona_groups.md)

## Escalate To

- [council_discuss.md](council_discuss.md) when the user needs iterative back-and-forth.
