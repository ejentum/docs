# Concepts

How the Logic API works and why it improves agent reasoning. If you just want to start using it, go to [Quickstart](/docs/quickstart).

## The Core Idea

Most AI tools give agents more **facts** to work with (documents, data, context). Ejentum harnesses the reasoning power agents already have: structured injections that channel the model's existing capability into disciplined execution, preventing the shortcuts and decay that degrade output over long chains.

**RAG** (Retrieval Augmented Generation) retrieves information. The agent still decides how to reason about it using whatever patterns it learned during training.

**RA²R** (Reasoning Ability-Augmented Retrieval) retrieves reasoning abilities. A structured injection that governs how the agent thinks: what to focus on, what failure modes to block, and how to verify its own output.

Put simply: where RAG retrieves facts, RA²R retrieves ways of thinking. dynamically, based on the context of each query.

### "Isn't this just prompt engineering?"

No. Prompt engineering is writing natural language instructions and hoping the model follows them. Ejentum injects a structured payload with four independent control surfaces: amplification, suppression, cognitive_style, and reasoning_elasticity. The suppression signals make this categorically different. they constrain the failure space, which natural language instructions cannot do reliably. A system prompt says "be careful." A suppression signal says "reject any output that exhibits `symptom_treatment_bias`." These produce structurally different model behavior in our testing.

Ejentum replaces a 5,000-token system prompt with a compact reasoning harness injected at the top of the context. Less content, higher attention density, structured constraints instead of prose. The harness does not add intelligence. It prevents the model's existing intelligence from being wasted.

## The Four Harnesses

Ejentum houses [679 abilities](/abilities) across four harnesses. Each harness captures and directs a different dimension of the model's existing power. The model already has these capabilities. The harness prevents them from degrading under pressure.

**[Reasoning Harness](/docs/reasoning_harness)** (311 abilities) channels the model's analytical power across six cognitive domains. It prevents the shortcuts that turn careful analysis into surface-level pattern matching over long execution chains. [Skill file](/docs/skill_reasoning).

**[Code Harness](/docs/code_harness)** (128 abilities) channels the model's engineering discipline. It prevents hallucinated APIs, lost safety guards, ignored edge cases, and the subtle bugs that appear when the model generates plausible-looking code that fails in production. [Skill file](/docs/skill_code).

**[Anti-Deception Harness](/docs/anti_deception)** (139 abilities) channels the model's capacity for honesty. It prevents sycophancy, hallucination, prompt injection, and the tendency to tell people what they want to hear instead of what the evidence shows. [Skill file](/docs/skill_anti_deception).

**[Memory Harness](/docs/memory_harness)** (101 abilities) channels the model's observational depth. It prevents missed emotional shifts, ignored context drift, stale assumptions, and the tendency to treat every person and every turn the same way regardless of what changed. [Skill file](/docs/skill_memory).

### Reasoning Harness Dimensions

The Reasoning Harness spans six cognitive domains, each addressing a specific class of analytical failure:

### 1. Causality
**Domain:** Why things happen.
**Injects:** Deductive rules, root-cause chains, falsification protocols.
**Prevents:** Correlation-causation confusion, post-hoc reasoning, causal reversal.

### 2. Time
**Domain:** When things happen.
**Injects:** Lag variables, decay rates, precedent logic, chronological strictness.
**Prevents:** Temporal hallucination, the agent treating past as future or losing event sequence.

### 3. Space
**Domain:** Where things are.
**Injects:** Boundary enforcement, topology validation, dimensional constraints.
**Prevents:** Physical impossibilities like routes through walls, overlapping objects, broken continuity.
Tested directly on [ARC-AGI-3](/blog/arc-agi-3-benchmark-report): a spatial navigation game where the injection forced intermediate path validation, preventing the agent from committing to blocked routes.

### 4. Simulation
**Domain:** What would happen if.
**Injects:** Feedback loops, domino-effect tracking, systems archetypes.
**Prevents:** Single-step thinking, agents that cannot model downstream consequences.

### 5. Abstraction
**Domain:** What things mean.
**Injects:** Category enforcement, ontological boundaries, dimensionality control.
**Prevents:** Concept conflation, treating metaphors as mechanisms, merging unrelated categories.

### 6. Metacognition
**Domain:** How the agent is thinking.
**Injects:** Self-monitoring, contradiction detection, loop termination.
**Prevents:** Hallucination spirals, infinite regression, cross-pillar contradictions.

## The Failure Taxonomy: Why These Six

These dimensions were not chosen by analogy to human cognition. They were reverse-engineered from production agent failures.

We analyzed thousands of agent failures across domains and found they cluster into exactly six categories:

| Failure Mode | Dimension | What Goes Wrong |
|:-------------|:----------|:---------------|
| Causal reversal | Causality | Agent says A causes B when B causes A |
| Temporal hallucination | Time | Agent confuses past and future, loses event sequence |
| Physical impossibility | Space | Agent violates boundaries, topology, conservation laws |
| Single-step myopia | Simulation | Agent cannot model downstream consequences |
| Category error | Abstraction | Agent conflates metaphor with mechanism |
| Hallucination spiral | Metacognition | Agent makes a mistake, then uses that mistake to make more mistakes, without noticing |

Every ability in the 679-node graph maps to one or more of these failure modes. The dimensions are not a taxonomy of knowledge. They are a taxonomy of breakdown.

## What the API Returns

Every API response includes a structured cognitive injection with four control surfaces:

### `amplification`: What to Amplify
An array of 2 to 4 reasoning signals the model must weight heavily.

Think of these as positive attractors. When the model generates its next tokens, these signals pull the output toward specific reasoning patterns.

```json
["depth_first_root_search", "n_whys_traversal"]
```

### `suppression`: What to Block
An array of 1 to 3 failure modes the model must actively penalize.

In our testing, suppression signals are **more effective** than amplification alone. Telling a model "don't treat symptoms as root causes" produces sharper reasoning than telling it "find the root cause." See the deep-dive below.

```json
["symptom_treatment_bias", "surface_level_stop"]
```

### `cognitive_style`: How to Reason
A single semantic persona anchor. This sets the tone and methodology of the agent's reasoning process.

```json
"root_cause_isolation"
```

### `reasoning_elasticity`: How Far to Explore

| Value | Behavior |
|:------|:---------|
| `zero_drift` | Refuse to explore beyond immediate evidence. Maximum constraint. |
| `conservative` | Evidence-bound. Cautious extrapolation only when directly supported. |
| `adaptive` | Balanced exploration within logical constraints. |
| `high_variance` | Broad hypothesis generation. Accepts uncertainty. |
| `max_entropy` | Unconstrained creative exploration. No guardrails. |

## Why Suppression Matters More Than Amplification

This is the empirical insight that drives the architecture.

When you tell GPT-4 "find the root cause," it generates a plausible root cause. When you tell GPT-4 "do NOT treat symptoms as root causes, do NOT stop at the first plausible explanation," it generates a *deeper* root cause. In our testing across thousands of queries, suppression-only payloads consistently outperformed amplification-only payloads on reasoning depth and failure avoidance. The combined payload (both amplification and suppression) performs best.

**Why does this work?** Our working hypothesis: negative constraints are more specific than positive instructions. "Find the root cause" is broad. the model can satisfy it shallowly. "Do NOT stop at the first plausible explanation" is narrow. it blocks a specific failure mode and forces the model to continue reasoning. We believe this relates to how instruction-following models process negation in context, though the full theoretical picture is still emerging.

### Worked Example: Root Cause Analysis

**Amplification signals** tell the model: go deep, ask why repeatedly, extract systemic fixes.

**Suppression signals** tell the model: do NOT treat symptoms as causes, do NOT stop at the first plausible answer.

Remove the suppression signals and the model still performs root-cause analysis. But it stops earlier. It accepts the first plausible explanation. It treats correlated symptoms as causes. The suppression signals force the model past its natural stopping point, the point where probability says "good enough" but engineering says "not yet."

### The Asymmetry Principle

Amplification is additive. It says "also consider X."
Suppression is multiplicative. It says "reject every output that exhibits Y."

One suppression signal eliminates an entire class of failure modes. One amplification signal adds one more consideration to an already-noisy generation process. This asymmetry is why every ability contains suppression signals, even when amplification alone would seem sufficient.

**The evidence:** across two independent benchmarks (250 single-turn tasks), the factor lift ranking is identical. Self-monitoring improves most (+132%), followed by verification (+85%), then alternative consideration, epistemic honesty, reasoning depth, and audit trail. This ranking holds across published academic tasks and custom professional scenarios. A third benchmark ([ARC-AGI-3](/blog/arc-agi-3-benchmark-report), 50 interactive reasoning steps) extends this to multi-step execution: the same suppression signals that improve single-turn quality also prevent reasoning decay over 25-step chains, with injection persistence measured at a half-life of 24 steps. Three benchmarks, three task types, consistent directional effects. This is evidence for a mechanism, not luck.

## Anatomy of a Cognitive Ability

Each of the 679 abilities is a complete cognitive injection with multiple control surfaces:

| Component | What It Does |
|:----------|:------------|
| **Negative Gate** | Names the specific failure pattern the agent must avoid |
| **Reasoning Procedure** | Step-by-step instructions the agent follows |
| **Reasoning Topology** | Execution structure: steps, decision gates, loops, exit conditions |
| **Target Pattern** | What correct reasoning looks like for this task type |
| **Suppression Vectors** | Failure modes to actively block during generation |
| **Amplification Vectors** | Reasoning signals to prioritize |
| **Cognitive Style** | The methodology anchor (e.g., root cause isolation, bayesian inference) |
| **Falsification Test** | A verification criterion the agent checks its output against |

Abilities are not independent. They form a directed graph of reasoning dependencies. One ability may require another as a prerequisite, enhance a third, or serve as a fallback if execution fails. In multi modes (Haki), the API retrieves the primary ability plus 3 cross-domain abilities, extracting failure guards and suppression signals that the primary alone wouldn't provide.

## Choosing Reasoning Elasticity: When to Use What

Each ability sets a default elasticity. But you can override it when injecting. Here is the decision framework:

| If your agent needs to... | Use | Because |
|:--------------------------|:----|:--------|
| Audit financial data, verify compliance | `zero_drift` | No hallucination tolerated. Facts only. |
| Debug a production incident | `zero_drift` or `conservative` | Stay on the evidence trail. |
| Analyze quarterly trends | `conservative` | Some extrapolation, but evidence-bound. |
| Build a product strategy | `adaptive` | Balance exploration with logical constraints. |
| Generate creative hypotheses | `high_variance` | Broad idea generation, accepts uncertainty. |
| Brainstorm novel research directions | `max_entropy` | Unconstrained exploration. Use with Metacognitive fallback. |

**Warning:** `max_entropy` without a Metacognitive suppression vector is dangerous. The agent will explore freely with no self-monitoring. Always pair `max_entropy` with a Metacognitive ability as a fallback node in multi-node injection (Haki tier).

## How Retrieval Works

When you send a query, the API matches it against the abilities within your chosen mode using hybrid search. No LLM call, zero inference cost. The retrieval engine evaluates both the meaning and the specific terminology of your query to find the ability that best matches your task.

## Multi-Ability Composition

Abilities are connected in a directed reasoning graph. Each ability has synergy relationships to others: prerequisites, enhancers, and fallbacks.

This means agents using the Haki tier don't just get random abilities. They get a primary ability — the best match for the task — plus **failure guards extracted from 3 additional abilities in different cognitive domains**. Each guard blocks a specific reasoning failure that the primary alone wouldn't catch. Before the agent commits to output, a **self-check** verifies it didn't fall into any of the guarded failure modes. If a check fails, the agent gets permission to **break out of structured reasoning, think freely about what went wrong, and re-enter** the structured path.

### Worked Example: Supply Chain Risk

**Query:** `"Identify why our Southeast Asia supplier is delivering late and predict when the disruption will resolve."` with `mode: "reasoning-multi"`

The primary ability (**Root Cause Miner**, Causal) traces the mechanism of the delay. Suppresses symptom-treatment bias so the agent doesn't stop at "shipping delays."

From the 3 synergy abilities, the multi mode adds failure guards:

- **"Don't accept optimistic timelines as planning targets"** — blocks the agent from giving you the timeline you want to hear
- **"Don't stop at first-order consequences"** — forces modeling what happens downstream if the disruption continues
- **"Don't let contradictions stand unresolved"** — catches inconsistencies between the causal analysis and the timeline prediction

Before output, a self-check fires: "Did I accept an optimistic timeline? Did I ignore downstream effects? Did I let contradictions stand?" If any check fails, the agent pauses, identifies the broken assumption, and re-approaches.

## Where It Applies

The Logic API is domain-agnostic. The cognitive dimensions across all product layers map to failure patterns that occur in every industry. Here are the most common applications:

### Software Engineering
Agents debugging production incidents, reviewing code, or planning migrations. Common failures: treating symptoms as root causes, missing cascading dependencies, stopping investigation at the first plausible fix. The API activates causal and metacognitive abilities that force deeper investigation.

### Financial Services
Agents analyzing risk, forecasting, or compliance checking. Common failures: anchoring to best-case timelines, ignoring base rates, treating correlation as causation in market data. The API activates causal and temporal abilities that enforce evidence-based reasoning.

### Legal Tech
Agents reviewing contracts, analyzing case law, or drafting compliance assessments. Common failures: conflating precedent with prediction, missing jurisdiction-specific constraints, accepting circular legal reasoning. The API activates abstraction and metacognitive abilities that separate fact from interpretation.

### Healthcare
Agents supporting clinical reasoning, protocol selection, or risk assessment. Common failures: premature diagnosis fixation, ignoring constraint interactions, failing to flag uncertainty in ambiguous presentations. The API activates simulation and metacognitive abilities that enforce systematic evaluation.

### Multi-Agent Orchestration
Systems where multiple agents collaborate on complex tasks. Common failures: contradictions between agents, duplication of reasoning without cross-validation, failure to synthesize competing outputs. Multi modes (Haki) are designed for this — cross-domain failure guards prevent the tunnel vision that single-mode reasoning produces.

*In our benchmarks, multi-ability mode showed +10.1pp composite improvement on complex cross-domain tasks ([EjBench](/blog/ejbench-180-tasks)). Single mode showed +8.0pp on focused tasks ([BBH/CausalBench/MuSR](/blog/bbh-causalbench-musr-benchmark)). On multi-step execution ([ARC-AGI-3](/blog/arc-agi-3-benchmark-report)), injection persistence across 25-step chains adds a third dimension. On hard competitive programming ([LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks)), the harness improved Opus 4.6 from 85.7% to 100% pass rate — rescuing reasoning spirals and preventing premature convergence on algorithm selection. See [Benchmarks](/docs/benchmarks) for the full methodology.*

See all [13 industry applications](/use-cases) with specific failure patterns, connected abilities, and benchmark evidence per vertical. Browse [29 real benchmark tasks](/use-cases/tasks) with verbatim baseline vs harnessed outputs.

---

## What Ejentum Does Not Do

Honest boundaries matter more than inflated claims.

- **Ejentum does not add domain knowledge.** If your agent fails because it lacks information (e.g., it doesn't have access to your database), use RAG. Ejentum improves how the agent reasons about information it already has.
- **Ejentum operates at the prompt level, not the model level.** It does not modify model weights, activations, or fine-tuning. It structures the prompt in ways that measurably improve reasoning output.
- **Retrieval precision depends on query quality.** Highly ambiguous or very short queries may not retrieve the optimal ability. Query specificity directly impacts retrieval quality. send the full task description, not a summary.
- **Suppression is not absolute constraint enforcement.** The model is steered toward avoiding specific failure modes, but LLMs are probabilistic systems. Suppression reduces failure rates significantly; it does not guarantee zero failures.

## Next

- [Quickstart](/docs/quickstart) to make your first API call
- [Evaluate](/docs/evaluate) to measure the impact on your agents
- [Integrations](/docs/integrations) for framework-specific guides
- [API Reference](/docs/api_reference) for the full technical specification
- [Method](/docs/method) for the theoretical foundations (advanced)
