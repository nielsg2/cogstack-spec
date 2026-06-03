# CogStack — Technical Specification

CogStack is a two-layer architecture for autonomous AI agent systems:

1. **Persistent cognition substrate** — externalizes the context produced by
   interactive AI sessions, compresses it into a durable memory store, and
   rehydrates it into subsequent API calls so agents reason continuously
   across sessions and execution environments.

2. **Multi-vendor cognitive stacking** — orchestrates agents from different
   providers and model families so the composite system exhibits properties
   that no single constituent produces alone, with agent diversity selected
   to maximize genuine independence rather than nominal redundancy.

AIDL (see [aidl-spec](https://github.com/nielsg2/aidl-spec)) provides the
governance grammar that bounds CogStack deployments: it defines agent roles,
accumulates per-agent reputation, and weights adjudication by provider
independence. CogStack is the orchestration layer that produces the inputs
AIDL consumes; the two are designed as matched layers of the same system.

---

## 1. Problem Statement

**Statefulness.** Current generative AI systems are fundamentally stateless.
A session produces rich context — decisions, rationale, queued tasks,
accumulated state — that evaporates when the session ends. API-driven task
runners are disconnected from this high-context interactive work. The result:
no agent can reason over time, maintain continuity between invocations, or
operate headlessly without a human re-establishing context at each call.

**Redundancy without diversity.** A naive approach to multi-agent reliability
is to run the same prompt through the same model multiple times and vote.
This achieves variance reduction but not genuine decorrelation: agents built
on the same base model share architecture, training corpus, post-training
shaping, and failure modes. Their agreement is correlated, not corroborating.
A system that cannot distinguish correlated agreement from genuine consensus
manufactures false confidence.

CogStack addresses both: the persistent cognition layer solves statefulness;
the cognitive-stacking layer solves redundancy-without-diversity.

---

## 2. Layer 1 — Persistent Cognition Substrate

### 2.1 Context Compression Engine

Receives the artifact of an interactive AI session — conversation history,
structured outputs, intent signals — and transforms it into **compact
semantic blocks**: a machine-readable representation of essential intent,
task state, and contextual constraints, small enough to inject into a future
API prompt and rich enough to restore operational reasoning.

Semantic blocks are typed (task, decision, constraint, background,
open-question) and tagged with confidence, recency, and relevance signals.
Stale or low-confidence blocks are trimmed at rehydration time. Compression
is intentionally lossy; verbatim transcript recovery is not a goal.

### 2.2 Memory Store

A durable repository of semantic blocks, execution records, and directive
state. Format-agnostic by design — implementations may use structured log
files, a graph model (nodes for tasks, decisions, agents; edges for
dependencies and ownership), or a vector index for embedding-based retrieval.
The defining property is durability across process restarts, machine reboots,
and environment changes.

### 2.3 Rehydration Pipeline

At execution time, constructs an enriched API prompt by:

1. Reading the next directive from the queue.
2. Querying the Memory Store for contextually relevant semantic blocks.
3. Ranking and trimming blocks to fit within the target model's context
   budget.
4. Composing the prompt: system instructions + rehydrated context + current
   task directive.

Context budget is treated as a first-class constraint, not an overflow
condition. The pipeline never exceeds a model's context window; it trims
by relevance, recency, and confidence.

### 2.4 Execution Daemon (gpt-cogd)

The headless process that acts. The daemon:

- Polls the Directive Manager for executable tasks.
- Invokes the Rehydration Pipeline to construct an enriched prompt.
- Calls the target AI API.
- Records the response in the Memory Store (raw output and a compressed
  semantic block).
- Updates task state (completed, blocked, deferred).
- Appends every action to the Audit Log.

Designed for headless, long-running operation: no GUI required, no human
present at invocation, runnable on schedule or trigger.

### 2.5 Directive Manager

A task queue and scheduler. Each directive entry specifies:

- Task description (what to do).
- Preconditions (what must be true in the Memory Store before execution).
- Priority (ordering among ready tasks).
- Execution constraints (scope limits, output targets).
- Status (pending, ready, in-progress, completed, blocked, deferred).

Precondition evaluation reads from the Memory Store, enabling data-dependent
execution ordering without a separate workflow engine.

### 2.6 Audit Log

An append-only, versioned record of every execution event: task dispatched,
prompt constructed, API call made, response received, semantic block stored,
state change applied, error encountered. The log supports:

- Governance review (did the agent behave as directed?).
- Debugging (why did a task fail or produce unexpected output?).
- Reputation scoring (in AIDL-governed deployments).
- Rollback (reversing the effects of a bad execution).

### 2.7 Plugin / Extension Bus

A modular extension mechanism allowing new capabilities — additional tool
integrations, alternative memory backends, custom compression strategies, new
provider adapters — to be added without modifying the core. The bus is how a
deployment adapts CogStack to its environment (which models, which tools, which
storage) and how the orchestration layer is extended toward the embodiments in
§7 without forking the substrate. Plugins are subject to the same audit and
write-scope discipline as the core.

---

## 3. Layer 2 — Multi-Vendor Cognitive Stacking

### 3.1 The Decorrelation Premise

Running the same prompt through N instances of the same model is *not* a
cognitive stack — it is repeated sampling from the same distribution. The
genuine gain from multi-agent systems comes from **decorrelated diversity**:
agents whose training corpora, architectures, and post-training shaping differ
enough that their failure modes do not coincide. Where one model has a
structural blind spot, another's training may cover it.

Selecting agents for a stack on the basis of nominal vendor diversity (three
different logos) can still yield correlated outputs if those vendors share
base models, training pipelines, or RLHF vendors. A CogStack deployment
selects agents with the explicit intent to maximize genuine independence, not
nominal headcount.

### 3.2 Composition Topologies

CogStack supports four composition patterns; deployments choose based on task
type and stakes:

**Parallel-and-adjudicate.** All agents receive the same task independently
and produce outputs simultaneously. An adjudication step (governed by AIDL's
reputation-weighted voting) resolves disagreements. Best for tasks where
correctness can be compared across outputs — code generation, factual
retrieval, structured extraction.

**Sequential / pipeline.** Agent A drafts; Agent B critiques; Agent C
synthesizes. Roles specialize by stage. Best for tasks where critique
benefits from seeing a first attempt — document review, policy analysis,
complex reasoning chains.

**Adversarial.** One or more agents are explicitly tasked to find flaws in
the primary agent's output. Disagreement is the designed product, not a
failure to resolve. Best for high-stakes outputs where sycophantic agreement
is a risk — security review, compliance analysis, argument stress-testing.

**Router / dispatch.** Task type determines which agent handles which
subtask, based on a profile of each model's known strengths and failure
modes. Best for heterogeneous workloads where no single model dominates
across all task categories.

Deployments may compose topologies (e.g., parallel-and-adjudicate for the
critique stage of a pipeline, adversarial for the final sign-off).

### 3.3 When to Stack

Cognitive stacking multiplies API cost and latency. It pays for itself on:

- **High-stakes outputs** — where a wrong answer has significant downstream
  consequence.
- **Correctness-critical tasks** — where there is a ground truth and errors
  are detectable.
- **High-ambiguity inputs** — where multiple valid interpretations exist and
  resolving ambiguity early prevents expensive rework.
- **Anti-sycophancy requirements** — where a single model's agreement with
  the user's framing is a known risk.

For the vast majority of tasks, a single well-prompted model is correct and
stacking is unnecessary overhead. A CogStack deployment names the criteria
under which stacking is invoked; it is not the default execution path.

### 3.4 Error Modes Specific to Stacking

Stacking can amplify rather than cancel error if not properly governed:

- **Correlated consensus laundering** — if a majority of stacked agents
  share a structural bias, the stack can produce a confident-sounding wrong
  answer that appears better-validated than any single agent would have.
- **Adjudication layer bias** — if the mechanism that resolves disagreement
  is itself biased, diversity in the stack is negated at the synthesis step.

This is the precise reason CogStack is designed as the orchestration
complement to AIDL's governance layer, not as a standalone system. AIDL's
reputation weighting and provider-independence discounting are the guardrails
that prevent the stack from manufacturing false confidence.

---

## 4. AIDL Integration

CogStack and AIDL are complementary layers:

| Layer | System | Function |
|-------|--------|----------|
| Orchestration | CogStack | Assembles agents, manages memory, executes directives |
| Governance | AIDL | Defines roles, accumulates reputation, adjudicates conflicts |

CogStack dispatches tasks; AIDL governs how disputes between agents are
resolved and how agent authority evolves with observed behavior. A deployment
without AIDL's governance has no principled mechanism for weighting
disagreements or detecting agent degradation; a deployment without CogStack's
persistent memory has no continuity across sessions. The layers are
independently deployable but designed for joint use.

---

## 5. Data Flow

```
Interactive session
       │
       ▼
Context Compression Engine ──► Memory Store ◄── prior sessions
                                    │
                                    ▼
                            Directive Manager (task queue)
                                    │
                                    ▼
                            Rehydration Pipeline
                            (query → rank → trim → compose prompt)
                                    │
                                    ▼
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
               Agent A          Agent B         Agent C
             (Provider X)    (Provider Y)    (Provider Z)
                    └───────────────┼───────────────┘
                                    ▼
                          AIDL Adjudication Layer
                          (reputation-weighted, independence-discounted)
                                    │
                                    ▼
                              Audit Log (append-only)
                              Memory Store (update)
```

---

## 6. Reference Implementation Notes

The reference implementation used PowerShell for the execution daemon
(`gpt-cogd`) on Windows, with a JSON-based task queue
(`Job_Queue/`) and a file-system-backed memory store. The original
implementation began as a single-vendor wrapper and generalized to the
vendor-agnostic, multi-provider model described here. The architecture is
deliberately language- and platform-agnostic; the PowerShell implementation
is illustrative, not prescriptive. Any language capable of making HTTP calls,
reading and writing structured files, and running as a background process can
implement this specification.

---

## 7. Disclosed Future Embodiments

The following embodiments are disclosed as intended directions, both to document
design intent and to broaden the prior-art footprint of this specification:

- **Dual-mode (voice + text) interaction** — text-to-speech output rendering
  with a synchronized live transcript, so the same persistent-cognition loop is
  driven by voice or text interchangeably.
- **API chaining and orchestration** — composing multiple external tools and
  services into a single directive's execution, with the Memory Store carrying
  intermediate state across the chain.
- **Autonomous agent / task queue with memory injection** — long-running
  autonomous tasks that carry persistent context forward across invocations
  rather than starting cold.
- **Secure execution sandbox** — a constrained execution boundary (sandboxed
  VM or equivalent) for AI-directed automation, so actions taken by the daemon
  are contained and reversible.
- **Cross-vendor consensus and contradiction detection** — routing logic across
  multiple model providers with explicit detection of agreement and
  disagreement between models (the operational core of cognitive stacking,
  §3), plus embedding-indexed memory of recurring patterns and operator
  preferences, using local inference where appropriate and cloud inference
  where necessary.
