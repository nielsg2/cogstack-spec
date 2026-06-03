# Provenance & Open-Publication Statement

## Origin

CogStack began as a personal engineering effort to solve a specific problem:
generative AI systems are stateless, but autonomous agents need to reason
continuously. The solution was to externalize the context — compress it,
store it durably, and rehydrate it into subsequent API calls — and to
orchestrate multiple agents across providers so the composite exhibits
properties none of the constituents produce alone.

Its development is recorded in timestamped version-control history:

- **2025-06-08** — the persistent-AI-cognition seed first appears under version
  control: a structured AI-interaction history archive and AI "session
  continuity" — the externalized memory and continuity that became CogStack's
  cognition substrate. This is the same day the companion AIDL directive
  framework first appears; the two were conceived together.
- **2025-06-09** — an AI session-writer and session-closer (prompt execution
  and session-closure logging/archival), a "Structured AI Workflow Continuity"
  document, and a task/refactor queue are added — the daemon, audit-log, and
  queue scaffolding.
- **2025-06-18** — execution bridge and logging framework consolidated.
- **2025-06-19** — the memory-rehydration mechanism is recorded; the name
  "CogStack" comes into use and a dedicated repository is initialized
  ("CogStack: AI Daemon & Workflow Orchestration").
- **2025-06-23** — CogStack is integrated as the cognition module of the
  persistent daemon.
- **2025-08-10** — architecture-considerations document added.

These dates are recorded in source-control history independently of this
publication.

This repository is a **curated public extract** of that larger private working
set. It contains only the general, reusable specification and architectural
patterns; project-specific, personal, and unrelated material has been
deliberately omitted. That is why this repository's own creation date is later
than the origin dates above — the *work* is from 2025; this *clean release*
is the public form of it.

## Relationship to AIDL

CogStack did not originate independently of AIDL — they were developed
together, in the same period, as two layers of the same architectural
insight. AIDL (first committed 2025-06-08, language named 2025-06-25)
provides the governance grammar that bounds agent behavior; CogStack provides
the orchestration pattern that assembles and executes agents. AIDL's
public specification is archived separately at
[github.com/nielsg2/aidl-spec](https://github.com/nielsg2/aidl-spec).

## On independent / parallel development

Through 2025–2026, the field converged rapidly on multi-agent orchestration
and persistent AI memory — LangChain, AutoGen, CrewAI, and similar frameworks
address related problems from different angles. CogStack's specific
contribution is the combination of a persistent-memory substrate with a
multi-vendor cognitive-stacking model that prioritizes decorrelated agent
diversity. No claim of primacy is made; the work is published openly as a
contribution and as prior art.

## Why this is published openly

This is released as a **contribution and as prior art**, under the MIT
license — not as a patent claim. Publishing openly:

- puts the work in the public record under its author's name,
- keeps the underlying ideas free for anyone to build on, and
- is consistent with the belief that infrastructure for governing AI agents
  responsibly should not be anyone's private monopoly.

## Author

Niels Goldstein. Independent work. Offered freely to anyone building
multi-agent systems who needs a starting point for persistent cognition and
decorrelated orchestration.
