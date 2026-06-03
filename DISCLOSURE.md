# Defensive Publication — CogStack: Persistent AI Cognition and Multi-Vendor Cognitive Stacking

**Author:** Niels Goldstein
**Publication date:** 2026-06-03
**License:** MIT
**Canonical repository:** https://github.com/nielsg2/cogstack-spec
**Status:** Defensive publication. This document is released to establish prior art and to contribute the described methods freely to the public. **No patent is claimed.**
**Scope:** This disclosure concerns only the persistent-cognition and multi-agent orchestration architecture described herein, and should not be construed as describing, disclosing, or relating to any other system or subject matter.

---

## Abstract

This disclosure describes **CogStack** — a two-layer architecture for autonomous AI agent systems. The first layer, the *persistent cognition substrate*, externalizes the context produced by interactive AI sessions, compresses it into a durable memory store, and rehydrates it into subsequent stateless API calls, enabling agents to reason continuously across sessions and execution environments. The second layer, *multi-vendor cognitive stacking*, orchestrates agents drawn from different model providers and families so the composite system exhibits emergent properties that no single constituent produces alone — specifically, decorrelated diversity in which each agent's failure modes do not coincide with the others'. CogStack is designed as the orchestration complement to AIDL (AI Directive Language; see companion disclosure), which provides the governance grammar that bounds agent behavior and weights adjudication by provider independence. Together they form a matched two-layer system: CogStack orchestrates decorrelated cognition; AIDL governs and adjudicates it.

---

## 1. Field and Background

As AI agents are increasingly delegated authority to take consequential actions, two structural deficiencies limit their reliability.

**Statefulness.** Generative AI systems are stateless. Interactive sessions produce rich context — decisions, rationale, queued tasks, accumulated state — that evaporates at session end. API-driven automation is disconnected from this high-context work. The result is that agents cannot reason continuously over time, cannot resume interrupted work, and cannot operate headlessly without a human re-establishing context at each invocation.

**Redundancy without diversity.** A common attempt at multi-agent reliability is to run the same prompt through multiple instances of the same model and vote. This achieves variance reduction but not genuine decorrelation: agents built on the same base model share architecture, training corpus, post-training shaping, and failure modes. Their agreement is correlated rather than corroborating. A system that cannot distinguish correlated agreement from genuine independent consensus manufactures false confidence.

CogStack addresses both deficiencies with two cooperating architectural layers.

---

## 2. Layer 1 — Persistent Cognition Substrate

### 2.1 Context Compression Engine

Receives the artifact of an interactive AI session and transforms it into **compact semantic blocks**: a typed, tagged, machine-readable representation of essential intent, task state, and contextual constraints, sized to fit within a future API prompt. Block types include task, decision, constraint, background, and open-question. Each block carries confidence, recency, and relevance signals that govern trimming at rehydration time. Compression is intentionally lossy; the target is operational continuity, not verbatim recovery.

### 2.2 Memory Store

A durable repository of semantic blocks, execution records, and directive state, persisting across process restarts and environment changes. Implementation format is agnostic — structured log files, graph models (nodes for tasks, decisions, and agents; edges for dependencies and ownership), or vector indices for embedding-based retrieval are all valid substrates. Durability is the defining constraint.

### 2.3 Rehydration Pipeline

At execution time, constructs an enriched API prompt by: (1) reading the next directive from the queue; (2) querying the Memory Store for contextually relevant semantic blocks; (3) ranking and trimming blocks to fit the target model's context budget; (4) composing the prompt from system instructions, rehydrated context, and the current task directive. Context budget is a hard constraint, not an overflow condition; the pipeline never exceeds the model's window.

### 2.4 Execution Daemon

A headless background process that polls the Directive Manager for executable tasks, invokes the Rehydration Pipeline to construct enriched prompts, calls the target AI API, records responses in the Memory Store (as raw output and as a compressed semantic block), updates task state, and appends every action to the Audit Log. Designed for long-running, unattended operation without a GUI or human present at invocation.

### 2.5 Directive Manager

A task queue and scheduler in which each directive specifies a description, preconditions (evaluated against the Memory Store for data-dependent ordering), priority, execution constraints, and status. The Directive Manager surfaces the highest-priority ready task to the daemon; precondition evaluation enables conditional execution ordering without a separate workflow engine.

### 2.6 Audit Log

An append-only, versioned record of every execution event — task dispatch, prompt construction, API call, response, semantic block storage, state change, and error. The log supports governance review, debugging, reputation scoring (in AIDL-governed deployments), and rollback.

**2.7 Plugin / Extension Bus.** A modular extension mechanism by which new capabilities — tool integrations, alternative memory backends, custom compression strategies, additional provider adapters — are added without modifying the core, subject to the same audit and write-scope discipline as the core.

---

## 3. Layer 2 — Multi-Vendor Cognitive Stacking

### 3.1 Decorrelated Diversity

The genuine gain from multi-agent systems requires agents whose training corpora, architectures, and post-training shaping differ enough that their failure modes do not coincide. Where one model has a structural blind spot, another's training may cover it. Nominal vendor diversity (different logos, shared base models) does not achieve this; decorrelated diversity requires deliberate selection of agents whose underlying characteristics are genuinely independent.

### 3.2 Composition Topologies

CogStack defines four composition patterns:

**Parallel-and-adjudicate.** All agents receive the same task independently and produce outputs simultaneously; an adjudication step (governed by AIDL's reputation-weighted voting) resolves disagreements. Suited to tasks where correctness can be compared across outputs.

**Sequential / pipeline.** Agents specialize by stage — draft, critique, synthesize — with each stage's output feeding the next. Suited to tasks where critique benefits from seeing a first attempt.

**Adversarial.** One or more agents are explicitly tasked to find flaws in the primary output; disagreement is the designed product. Suited to high-stakes outputs where sycophantic agreement is a risk.

**Router / dispatch.** Task type determines agent assignment, based on a profile of each model's known strengths and failure modes. Suited to heterogeneous workloads where no single model dominates across task categories.

Topologies are composable; a deployment may apply parallel-and-adjudicate within the critique stage of a sequential pipeline.

### 3.3 Invocation Criteria

Cognitive stacking multiplies API cost and latency; it is not the default execution path. CogStack specifies invocation criteria: high-stakes outputs, correctness-critical tasks, high-ambiguity inputs, and anti-sycophancy requirements. A conforming deployment names the conditions under which stacking is triggered.

### 3.4 Error Modes and Governance Dependency

Stacking can amplify rather than cancel error when a majority of agents share a structural bias (correlated consensus laundering) or when the adjudication mechanism is itself biased (adjudication layer bias). These failure modes are why CogStack is designed as the orchestration complement to AIDL's governance layer: AIDL's provider-independence discounting and reputation weighting are the architectural guardrails that prevent the stack from laundering correlated agreement into false confidence.

---

## 4. AIDL Integration

| Layer | System | Function |
|-------|--------|----------|
| Orchestration | CogStack | Assembles agents, manages persistent memory, executes directives |
| Governance | AIDL | Defines roles, accumulates reputation, adjudicates conflicts |

CogStack dispatches tasks and manages context; AIDL governs how agent authority is assigned, how disputes are resolved, and how reputation evolves from observed behavior. The layers are independently deployable but designed for joint use; a deployment without governance has no principled mechanism for weighting disagreements or detecting agent degradation, and a deployment without persistent memory has no continuity across sessions.

---

## 5. Example Embodiment

Three agents — drawn from three different providers — are configured in a parallel-and-adjudicate topology for a correctness-critical code-review task. The Rehydration Pipeline constructs each agent's prompt from the shared task directive and individually relevant memory blocks drawn from the Memory Store. Each agent produces an output independently. Because all three are on different providers, AIDL's adjudication layer treats their agreement as more corroborating than it would for same-provider agents. One agent's output contradicts the others on a specific claim; the adversarial pattern is invoked for that claim, tasking a fourth agent to attempt to refute the primary finding. The result and the refutation attempt are logged; the Memory Store is updated with the final adjudicated output and a compressed semantic block representing the decision and rationale. The Directive Manager marks the task complete and advances to the next.

---

## 6. Reference Implementation

The reference implementation used PowerShell for the execution daemon on Windows, with a JSON-based task queue and a file-system-backed memory store. The persistent-AI-cognition substrate — a structured AI-interaction history archive and AI session continuity — first appears under version control on 2025-06-08 (the same day as the companion AIDL directive framework); session-writer/closer scaffolding, a workflow-continuity document, and a task queue follow on 2025-06-09; the system was named "CogStack" and given a dedicated repository on 2025-06-19, and integrated as the cognition module of a persistent daemon by 2025-06-23. The architecture is language- and platform-agnostic.

---

## 7. Disclosed Future Embodiments

The following embodiments are disclosed, both to document design intent and to broaden the prior-art footprint of this disclosure:

- **Dual-mode (voice + text) interaction** — text-to-speech output with a synchronized live transcript, driving the same persistent-cognition loop by voice or text interchangeably.
- **API chaining and orchestration** — composing multiple external tools and services into a single directive's execution, with the Memory Store carrying intermediate state across the chain.
- **Autonomous agent / task queue with memory injection** — long-running autonomous tasks that carry persistent context forward across invocations rather than starting cold.
- **Secure execution sandbox** — a constrained execution boundary (sandboxed VM or equivalent) for AI-directed automation, containing and making reversible the actions taken by the daemon.
- **Cross-vendor consensus and contradiction detection** — routing across multiple providers with explicit detection of agreement and disagreement between models (the operational core of cognitive stacking, §3), with embedding-indexed memory of recurring patterns and operator preferences, using local inference where appropriate and cloud inference where necessary.

## 8. Associated Terminology

The following terms are disclosed as descriptors of the methodology embodied by CogStack, to place them in the attributed public record:

- **Cognitive stacking** — composing agents from genuinely independent providers so the composite exhibits decorrelated diversity that no single constituent provides.
- **Semantic block** — a typed, tagged, compressed unit of session context (intent, decision, constraint) sized for re-injection into a future prompt.
- **Rehydration** — the explicit reconstruction of an agent's working context from durable memory at the start of an execution, as opposed to relying on a vendor's opaque native memory.
- **Cognitive instruction set** — a human-readable, machine-actionable directive package issued to an agent or daemon, functioning as an instruction set for autonomous execution.
- **Guard-railed task injection** — delivery of an isolated task payload with enforced constraints on output format, execution boundary, and error handling.

## 9. Statement of Defensive Publication

The author dedicates the methods described herein to the public as prior art. This disclosure is published openly under the MIT License at the canonical repository above. It is intended to ensure these methods remain freely available to all and to prevent their exclusive appropriation. No patent rights are asserted.
