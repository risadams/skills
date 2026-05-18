# Skill Graph

## Nodes

- `persona_consult`
- `council_consult`
- `council_define_personas`
- `council_define_persona_groups`
- `council_discuss`

## Edges

- `council_consult -> persona_consult` (fan-out to multiple personas)
- `council_consult -> council_define_personas` (load/override persona contracts)
- `council_consult -> council_define_persona_groups` (load/override persona groups)
- `council_discuss -> council_consult` (build structured answer per turn)
- `council_discuss -> persona_consult` (deep dive on one persona)
- `council_discuss -> council_define_persona_groups` (expand a small pre-made panel)
- `persona_consult -> council_discuss` (escalate when problem needs multi-turn)
- `council_define_personas -> persona_consult` (apply updated persona profile)
- `council_define_persona_groups -> council_consult` (apply updated group panel)
- `council_define_persona_groups -> council_discuss` (apply updated group panel)

## Suggested Execution Order

1. `council_define_personas` (optional)
2. `council_define_persona_groups` (optional)
3. `persona_consult` or `council_consult`
4. `council_discuss` when ambiguity or multi-turn debate is needed
