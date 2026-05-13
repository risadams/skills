# Clarity Council Markdown Skills

This is a simple, markdown-first skill suite.

No server runtime is required.
No MCP protocol is required.
Any LLM harness can consume these files as prompt skills.

## Start Here

- Skill graph: [SKILL_GRAPH.md](SKILL_GRAPH.md)
- Personas index: [personas/PERSONAS.md](personas/PERSONAS.md)

## Core Skills

- [persona_consult.md](persona_consult.md)
- [council_consult.md](council_consult.md)
- [council_define_personas.md](council_define_personas.md)
- [council_discuss.md](council_discuss.md)

## Interconnection Rule

Each skill references related skills under "Calls" and "Escalate To" so agents can chain behavior without custom runtime code.
