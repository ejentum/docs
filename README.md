# Ejentum Documentation

Everything you need to integrate Ejentum's Logic API into your agent: from a 30-second curl call to the theoretical foundations behind suppression-driven reasoning.

These are the same documents rendered at [ejentum.com/docs](https://ejentum.com/docs). Read them here or on the website: same content, your choice.

---

## What is RA²R?

Most AI tools give agents more **facts** to work with (documents, data, context). Ejentum harnesses the reasoning power agents already have: structured injections that channel the model's existing capability into disciplined execution, preventing the shortcuts and decay that degrade output over long chains.

**RA²R** (Reasoning Ability-Augmented Retrieval) retrieves reasoning abilities. A structured injection that governs how the agent thinks: what to focus on, what failure modes to block, and how to verify its own output. Where RAG retrieves facts, RA²R retrieves ways of thinking.

679 abilities across four harnesses. One REST endpoint. No SDK.

---

## Which doc do I need?

| I want to... | Read |
|--------------|------|
| Try it in 30 seconds | [Quickstart](getting-started/quickstart.md) |
| Test whether it works on my own tasks | [Evaluate](getting-started/evaluate.md) |
| Understand the mechanism | [Concepts](getting-started/concepts.md), then [The Method](understand/method.md) |
| See actual injection payloads | [Injection Examples](build/examples.md) |
| See before/after output quality | [Response Examples](build/response_examples.md) |
| Integrate with my framework | [Integrations](build/integrations.md) |
| Set up an agentic IDE (Cursor, Windsurf, Claude Code) | [Skill Files](#skill-files-agent-instructions) |
| Read the request/response schema | [API Reference](build/api_reference.md), then [Architecture](understand/architecture.md) |
| See benchmark results | [Benchmarks](understand/benchmarks.md) |
| Understand one specific product layer | [Reasoning](understand/reasoning_harness.md) · [Code](understand/code_harness.md) · [Anti-Deception](understand/anti_deception.md) · [Memory](understand/memory_harness.md) |
| Look up a specific term | [Glossary](reference/glossary.md) · [FAQ](reference/faq.md) |

---

## Getting Started

| Document | Description |
|----------|-------------|
| [Quickstart](getting-started/quickstart.md) | Add a cognitive injection to your agent. One API call. No SDK. |
| [Evaluate](getting-started/evaluate.md) | Measure whether Ejentum improves your agent. One afternoon. |
| [Concepts](getting-started/concepts.md) | How the Logic API works. Four product layers. Why suppression matters. |

## Product Layers

| Document | Description |
|----------|-------------|
| [Reasoning Harness](understand/reasoning_harness.md) | 311 abilities across 6 cognitive dimensions. The original product. |
| [Anti-Deception Harness](understand/anti_deception.md) | 139 abilities. Blocks sycophancy, hallucination, prompt injection, social engineering. |
| [Code Harness](understand/code_harness.md) | 128 abilities across 13 engineering disciplines. Code generation, refactoring, architecture. |
| [Memory Harness](understand/memory_harness.md) | 101 abilities. Perception sharpening, state tracking, behavioral calibration. |

## Build

| Document | Description |
|----------|-------------|
| [API Reference](build/api_reference.md) | Complete technical specification for the Logic API v1. |
| [Integrations](build/integrations.md) | n8n, LangChain, CrewAI, Claude SDK, Cursor, Make.com. |
| [Injection Examples](build/examples.md) | Real, complete injection payloads for all 7 modes. |
| [Response Examples](build/response_examples.md) | Real before/after outputs from blind benchmarks. |
| [Agent Tool Guide](build/agent_tool_guide.md) | Developer reference with code examples. |
| [Builder's Playbook](build/builders_playbook.md) | 28 screenshots from real work sessions. |

## Skill Files (agent instructions)

| Document | Description |
|----------|-------------|
| [Ejentum Skill File (all modes)](build/skill_unified.md) | Autonomous routing across all 4 harnesses. Drop-in for any agent. |
| [Reasoning Skill File](build/skill_reasoning.md) | 311 abilities, 6 dimensions, blind-evaluated. |
| [Code Skill File](build/skill_code.md) | 128 abilities, 13 disciplines, blind-evaluated. |
| [Anti-Deception Skill File](build/skill_anti_deception.md) | 139 abilities, 6 domains, blind-evaluated. |
| [Memory Skill File](build/skill_memory.md) | 101 abilities, two-pass protocol, blind-evaluated. |

## Understand

| Document | Description |
|----------|-------------|
| [The Method](understand/method.md) | Theoretical foundations. Failure dimensions. Suppression asymmetry. |
| [Benchmarks](understand/benchmarks.md) | Eight benchmark suites across four product layers. Full results. |
| [Architecture](understand/architecture.md) | Request lifecycle. What the API returns. How to inject. |

## Reference

| Document | Description |
|----------|-------------|
| [FAQ](reference/faq.md) | Straight answers. Compatibility, reliability, pricing, usage. |
| [Glossary](reference/glossary.md) | Key terms: Negative Gate, Reasoning Topology, Injection, Harness, Ability. |
| [Changelog](reference/changelog.md) | API version history and stability guarantees. |

---

## Links

- **Product:** [ejentum.com](https://ejentum.com)
- **Benchmarks:** [github.com/ejentum/benchmarks](https://github.com/ejentum/benchmarks)
- **Examples:** [github.com/ejentum/examples](https://github.com/ejentum/examples)
- **Blog:** [ejentum.com/blog](https://ejentum.com/blog)

## License

Documentation is released under [CC BY 4.0](LICENSE).
