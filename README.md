# CogStack — Persistent AI Cognition and Multi-Vendor Cognitive Stacking

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**A two-layer architecture for autonomous AI agent systems: a persistent cognition substrate that gives agents memory across sessions, and a multi-vendor cognitive stacking model that assembles agents from different providers to produce emergent reliability through decorrelated diversity.**

Current AI agents are stateless — context evaporates at session end. And naive multi-agent setups run the same model multiple times and call it redundancy, when what they've actually built is correlated agreement dressed up as consensus. CogStack addresses both: externalize and rehydrate the context so agents reason continuously, and stack agents from genuinely different providers so the composite covers failure modes no single constituent has.

## Contents

| File | What it is |
|------|------------|
| [`SPEC.md`](SPEC.md) | Technical architecture: persistent cognition substrate, cognitive stacking topologies, AIDL integration |
| [`DISCLOSURE.md`](DISCLOSURE.md) | Defensive publication document |
| [`PROVENANCE.md`](PROVENANCE.md) | Origin, timeline, and why this is published openly |

## The two layers

**Layer 1 — Persistent cognition substrate.** A Context Compression Engine compacts interactive session artifacts into typed semantic blocks. A Memory Store holds them durably. A Rehydration Pipeline injects relevant blocks into subsequent API prompts within the model's context budget. An Execution Daemon runs headlessly, polling a Directive Manager for tasks and appending every action to an Audit Log.

**Layer 2 — Multi-vendor cognitive stacking.** Agents are drawn from different providers and model families specifically to maximize genuine independence — not nominal vendor diversity. Four composition topologies are defined: parallel-and-adjudicate, sequential/pipeline, adversarial, and router/dispatch. Stacking is invoked selectively for high-stakes, correctness-critical, or anti-sycophancy-requiring tasks; it is not the default path.

## Relationship to AIDL and EthicalSwarm

CogStack is the **orchestration layer**. [AIDL](https://github.com/nielsg2/aidl-spec) provides the governance grammar that bounds it: agent roles, reputation accumulation, and adjudication weighted by provider independence. [EthicalSwarm](https://github.com/nielsg2/ethicalswarm-spec) provides the ethical governance framework for multi-agent deployments at scale.

The three form a matched set: CogStack orchestrates, AIDL governs, EthicalSwarm bounds the ethical perimeter.

## Status

Published openly as a contribution and as prior art, under the MIT license. This is a specification and reference design — not a product. A consolidated technical disclosure is archived as [`CogStack-Defensive-Publication.pdf`](CogStack-Defensive-Publication.pdf) and submitted as a defensive publication on the Technical Disclosure Commons.

## License

MIT. See [LICENSE](LICENSE).
