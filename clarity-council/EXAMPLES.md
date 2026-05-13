# Clarity Council Input Examples

This file contains input-only examples for invoking all major Clarity Council functionality.

## 1. Single Persona Consultation

Use when you want one focused perspective.

```json
{
  "skill": "persona_consult",
  "persona_name": "Senior Developer",
  "user_problem": "Our CI pipeline takes 45 minutes and blocks releases.",
  "context": "Monorepo with 12 services, mostly TypeScript, GitHub Actions.",
  "desired_outcome": "Reduce average pipeline time to under 15 minutes.",
  "constraints": [
    "No paid CI add-ons this quarter",
    "Must keep current required security scans"
  ],
  "depth": "standard"
}
```

## 2. Multi-Persona Council Synthesis

Use when you need tradeoffs across roles.

```json
{
  "skill": "council_consult",
  "user_problem": "Should we move from a modular monolith to microservices this year?",
  "context": "B2B SaaS, 20 engineers, rapid feature growth, occasional scaling pain.",
  "desired_outcome": "Choose architecture strategy for next 12 months.",
  "constraints": [
    "Cannot pause roadmap delivery",
    "Platform team has only 3 engineers"
  ],
  "selected_personas": [
    "Senior Architect",
    "Financial Officer",
    "DevOps Engineer",
    "Product Owner",
    "Devil’s Advocate"
  ],
  "depth": "deep"
}
```

## 3. Define Persona Overrides

Use when you want to tune persona behavior for the current session/workspace.

```json
{
  "skill": "council_define_personas",
  "overrides": {
    "Financial Officer": {
      "focus": [
        "cash flow impact",
        "run-rate stability",
        "payback period"
      ],
      "constraints": [
        "Flag recommendations with payback period over 12 months"
      ]
    },
    "Product Owner": {
      "soul": "Outcome-focused product strategist balancing customer value and delivery speed.",
      "constraints": [
        "Prefer options that preserve roadmap commitments in current quarter"
      ]
    }
  }
}
```

## 4. Multi-Turn Discussion (Initial Turn)

Use to begin an iterative clarification session.

```json
{
  "skill": "council_discuss",
  "requestText": "We need to cut cloud costs by 30% without hurting reliability. What should we do first?",
  "personasRequested": [
    "Ops Architect",
    "DevOps Engineer",
    "Financial Officer",
    "Security Expert"
  ]
}
```

## 5. Multi-Turn Discussion (Follow-Up Turn)

Use when continuing a prior discussion with returned state.

```json
{
  "skill": "council_discuss",
  "requestText": "Continue from prior turn.",
  "state": {
    "sessionId": "cc-session-2026-04-30-01",
    "turn": 2,
    "history": [
      "Turn 1: Council requested baseline cost breakdown by service and environment.",
      "Turn 2: User provided top 10 cost drivers and current SLO targets."
    ]
  },
  "answer": "We can rightsize Kubernetes nodes and reduce staging uptime from 24/7 to business hours."
}
```

## 6. Entry-Point Invocation Via SKILL.md Routing

Use when your harness invokes the root skill and lets it classify the mode.

```json
{
  "skill": "clarity-council",
  "user_problem": "We are debating SOC 2 Type II now vs next year while scaling enterprise sales.",
  "context": "Security team is 2 people; sales pipeline has 6 enterprise deals.",
  "desired_outcome": "Decide timing and scope with clear risks.",
  "constraints": [
    "No major hiring before Q4",
    "Avoid roadmap slip for existing contractual milestones"
  ],
  "selected_personas": [
    "Security Expert",
    "Financial Officer",
    "Growth Strategist",
    "Tech Lead"
  ],
  "depth": "standard"
}
```

## 7. Minimal Inputs (Fast Start)

Use smallest valid payload for quick checks.

```json
{
  "skill": "persona_consult",
  "persona_name": "Tech Lead",
  "user_problem": "Our sprint spillover is increasing."
}
```

```json
{
  "skill": "council_consult",
  "user_problem": "Should we sunset our legacy API version this quarter?"
}
```

```json
{
  "skill": "council_discuss",
  "requestText": "Help us choose between speed and reliability improvements this month."
}
```
