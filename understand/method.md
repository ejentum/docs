# The Method

Advanced reading. The theoretical foundations behind the Logic API. For practical usage, see the [Quickstart](/docs/quickstart). For the full benchmark data, see [Benchmarks](/docs/benchmarks).

Ejentum is built on a single claim: language model failures are structural, not informational. Models fail not because they lack knowledge, but because they take reasoning shortcuts under pressure. The solution is not to add more knowledge. It is to harness the intelligence the model already has, channeling it through structured cognitive operations that prevent decay over long execution chains. This page explains the science behind RA²R, the four harnesses (Reasoning, Code, Anti-Deception, Memory), and the design principles that make cognitive harnessing work.

---

## The Thesis

Retrieval-Augmented Generation solves an information problem. It gives the model facts it doesn't have. documents, context, grounded data. Ejentum solves a different problem: it harnesses the reasoning power the model already has. The model does not need more information. It needs the right cognitive operation activated at the right moment.

The distinction matters because most LLM failures in production are not information failures. A model with perfect recall can still:

- Reverse the direction of causality
- Confabulate plausible-but-fabricated timelines
- Generate unfalsifiable counterfactuals
- Collapse distinct categories into semantic blur
- Produce confident answers with no uncertainty signal

The failure is structural. The model applied the wrong cognitive operation. RAG can't fix this. it can only add more input to the same broken process.

**The RA²R hypothesis:** if you can identify the correct cognitive operation for a task type and inject it as structured context before generation begins, you shift the probability distribution over outputs in a measurable, predictable direction. The model does not need retraining. It needs a harness that channels its existing power into the right shape for the task at hand.

This is why injection is pre-generation. A post-hoc filter checks outputs after the damage is done. A pre-generation harness shapes what gets generated. The cognitive injection activates before the first token, channeling the model's power rather than adding to it.

---

## Six Reasoning Dimensions

The six dimensions are not a framework imposed on top of existing AI theory. They emerged from a taxonomy of how language models fail in production. organized by class of cognitive error, not by subject matter.

| Dimension | Core Failure |
|-----------|--------------|
| Causal | Direction-of-causation errors, correlation treated as cause |
| Temporal | Sequence errors, confabulated timelines |
| Spatial | Topology failures, boundary violations |
| Simulation | Counterfactual collapse, model-reality gap |
| Abstraction | Category collapse, over-generalization |
| Metacognition | Hallucination without recognition, bias blindness |

679 abilities are distributed across four product layers, each with its own cognitive dimensions.

A query is not classified into one dimension. It receives a continuous score across all six simultaneously. A supply chain risk query scores high on Causal and Temporal. A market expansion question scores on Spatial, Simulation, and Abstraction. The routing is probabilistic, not categorical. the vector engine finds cross-disciplinary intersections automatically.

### Causal (CA)

Addresses direction-of-causation errors. LLMs pattern-match on co-occurrence: when two events appear together in training data, the model treats temporal proximity as causal direction. Causal abilities suppress correlation-as-causation and enforce mechanistic audit trails.

### Temporal (TE)

Addresses timeline confabulation. Training data mixes epochs. models compress time, generating confident sequences from blended signal across different periods. Temporal abilities enforce sequence reconstruction and flag temporal gaps.

### Spatial (SP)

Addresses topology and boundary failures. LLMs have no persistent spatial representation. Containment, adjacency, and topology are implied from linguistic patterns, not computed. Spatial abilities enforce constraint-aware reasoning about physical and logical space.

### Simulation (SI)

Addresses counterfactual collapse. Counterfactual reasoning requires maintaining a modified world-state and reasoning within it. Models drift back toward the training distribution. the real world bleeds into the simulation. Simulation abilities hold the hypothetical stable.

### Abstraction (AB)

Addresses category collapse. Abstraction requires moving between levels of description without losing precision. LLMs over-generalize: specifics get absorbed into categories, categories lose their boundaries. Abstraction abilities maintain semantic resolution at every level.

### Metacognition (MC)

Addresses hallucination without recognition. Without self-monitoring, models generate fluent text that sounds like confident reasoning but contains no uncertainty signal. Metacognitive abilities inject calibrated uncertainty and flag low-confidence outputs explicitly.

---

## The Asymmetry Principle

Every ability carries two types of vectors: **amplification** and **suppression**. They are not symmetric in how they affect model behavior.

### Amplification

Positive attractors. The model is told what patterns to move toward. It can comply or drift. Amplification increases the probability of desirable outputs. but the model retains the ability to generate adjacent, undesirable ones.

- **Effect:** Additive
- **Vectors per ability:** 2–4
- **Example:** `["mechanistic_audit", "evidence_chain_tracing"]`

### Suppression

Named failure modes. When you explicitly name what the model must not do, its attention mechanism reallocates probability mass away from that region of the output space. The failure branch is not just deprioritized. it is structurally constrained.

- **Effect:** Multiplicative
- **Vectors per ability:** 1–2
- **Example:** `["shared_assumptions", "unverified_causal_claims"]`

### Why Suppression Compounds

Amplification operates on positive probability. it adds attractors to the generation process. Suppression operates on the constraint space. it removes failure regions before generation explores them. A constrained output space is smaller and more predictable than an attracted one.

In practice: a model told to "think deeply about causality" may or may not comply. A model told "never treat correlation as causation" changes its reasoning abilities. The instruction reaches a different layer of the generation process.

This is why suppression vectors are always 1–2 per ability while amplification can be 2–4. More amplification adds noise at the margin. More suppression compounds multiplicatively. each named failure mode eliminates an entire branch of incorrect outputs.

Direct evidence of persistence: on ARC-AGI-3 (an interactive reasoning benchmark where the agent takes 25+ sequential actions), a single suppression signal (`start_end_only_thinking`) remained active for 24 steps with no re-injection. The injection's half-life equaled the entire game session. Suppression does not fire once and decay. It persists as a structural constraint across extended execution chains. Full trace data in the [ARC-AGI-3 benchmark report](/blog/arc-agi-3-benchmark-report).

---

## Reasoning Elasticity

Not all tasks want the same kind of reasoning. Root-cause analysis requires tight constraint. the agent must stay close to evidence, refuse extrapolation, and flag gaps rather than fill them. Strategy ideation requires the opposite. broad hypothesis generation, tolerance for uncertainty, willingness to explore the improbable.

Reasoning elasticity is the control surface for this spectrum. It answers the question: *how far may the agent deviate from immediate evidence?*

### Two Control Levers

**`coherence_target`**. What the reasoning must remain anchored to.

| Value | Behavior |
|-------|----------|
| `causal_origin` | Reasoning traces backward to source events |
| `logical_consistency` | Maintains internal coherence across long chains |
| `creative_hypothesis` | Releases the anchor almost entirely |

**`expansion_factor`**. The radius of the exploration space.

| Value | Description |
|-------|-------------|
| `zero_drift` | Refuses to go beyond immediate facts. Maximum constraint. |
| `conservative` | Evidence-bound. Cautious extrapolation only. |
| `adaptive` | Balanced exploration within logical constraints. |
| `high_variance` | Broad hypothesis generation. Accepts uncertainty. |
| `max_entropy` | Unconstrained creative exploration. |

### Who Sets Elasticity

Each ability in the library has an embedded elasticity setting calibrated by the domain. Causal abilities use `zero_drift` by default. their purpose is precision. Simulation abilities use `high_variance`. their purpose is exploration. The agent does not configure this. The ability designer does.

This is a deliberate constraint. Agents should not be deciding how much to speculate. They should be executing a task with a well-defined epistemic posture. The elasticity setting is part of the ability's specification, not the agent's runtime choice.

---

## The Failure Taxonomy

The cognitive dimensions map to the most reproducible, high-severity failure modes in production LLM deployments. Each failure has a mechanism. a structural reason why language models fail this way reliably.

### Causal Reversal

**Mechanism:** LLMs pattern-match on correlation. When two events co-occur in training data, the model treats temporal proximity as causal direction. The output inverts the relationship: the effect becomes the cause.

**Consequence:** Root-cause analysis returns symptom treatment. Attribution errors compound downstream.

### Temporal Hallucination

**Mechanism:** Training data mixes epochs. Models compress time. events from different periods merge into coherent-sounding sequences. The model generates confident timelines from blended signal.

**Consequence:** Trend analysis returns fabricated trajectories. Planning built on hallucinated precedents fails silently.

### Boundary Violation

**Mechanism:** LLMs have no persistent spatial representation. Containment, adjacency, and topology are implied from linguistic patterns. not computed. Each token is predicted without a maintained coordinate system.

**Consequence:** Layout planning ignores physical constraints. Network topology analysis treats disconnected nodes as adjacent.

### Model-Reality Gap

**Mechanism:** Counterfactual reasoning requires holding a modified world-state and reasoning within it. LLMs drift back toward the training distribution. the real world bleeds into the simulated one. The model can't maintain the fiction.

**Consequence:** Stress tests return optimistic results. "What if" reasoning converges on "what is."

### Category Collapse

**Mechanism:** Abstraction requires moving between levels of description without losing precision at either level. LLMs over-generalize: specifics get absorbed into categories, and categories lose their boundaries. The result is semantic blur.

**Consequence:** Classification loses resolution at the boundary. Ontological reasoning breaks down at category edges.

### Hallucination Loops

**Mechanism:** Without a mechanism for self-monitoring, models generate fluent text that sounds like confident reasoning but contains no uncertainty signal. Errors are reinforced token-by-token.

**Consequence:** Confidence is uniform regardless of actual reliability. The agent produces outputs indistinguishable in tone from correct outputs. until verified externally.

---

## Anatomy of a Real Ability

Every ability is a structured node with triggers, control surfaces, and graph edges. Here is what a Causal ability for Bayesian inference contains:

| Component | What It Does |
|-----------|-------------|
| **Negative Gate** | Names the failure pattern: persisting with initial probability estimates despite contradicting evidence |
| **Reasoning Procedure** | Step-by-step: assign priors, accumulate evidence, update posteriors, verify calibration |
| **Suppression Vectors** | Block base-rate neglect, block anchoring to initial estimates |
| **Amplification Vectors** | Prioritize prior distribution weighting, posterior update cycles |
| **Cognitive Style** | Bayesian inference methodology |
| **Falsification Test** | "If the posterior was not updated in response to new evidence, calibration was skipped" |
| **Graph Edges** | Requires a Temporal ability (retroactive evidence re-evaluation). Enhances a Simulation ability (context modification tracking). |

This is not a flat prompt list. It is a directed graph of reasoning dependencies. Abilities connect across dimensions because real reasoning problems do not respect domain boundaries.

---

## Evidence: What the Benchmarks Show

### The Factor Ranking Is Consistent

Across three independent benchmarks (180 custom tasks, 70 published academic tasks, and ARC-AGI-3 interactive reasoning), the factor lift ranking on single-turn tasks is identical:

1. Self-monitoring (largest improvement)
2. Verification
3. Alternative consideration
4. Epistemic honesty
5. Reasoning depth
6. Audit trail
7. Correctness (flat or slight improvement)

Two independent task sets, different complexities, different sources, same ranking. A third benchmark, ARC-AGI-3 (interactive multi-step reasoning, 50 total steps across two conditions), adds multi-step evidence: the same suppression signals that improve single-turn quality also prevent reasoning decay over extended execution chains. Memory decay slope reversed from -0.005 (baseline, degrading) to +0.014 (augmented, improving).

A fourth benchmark — [LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks) (28 hard competitive programming tasks) — extends the evidence to code generation. The harness improved Opus 4.6 from 85.7% to 100% pass rate by preventing **reasoning spirals** (extended thinking consuming the full time budget without producing code) and **premature algorithmic convergence** (accepting a first-plausible solution that fails on edge cases). An independent blind evaluation confirmed the harness never loses on correctness (2-0) or robustness (4-0), with a 3.5x magnitude asymmetry in quality scores.

Four independent benchmarks, four different task types, consistent directional effects. See [Benchmarks](/docs/benchmarks) for the full comparison.

### Emergent Self-Monitoring

The single largest behavioral change: self-monitoring more than doubled across both benchmarks (+132% on published tasks, +92% on custom tasks).

This is not incremental. Agents shift from "I will answer the question" to "I will answer the question and explicitly check whether my reasoning is sound." The injections don't explicitly say "self-monitor." The suppression signals create checkpoints that force self-monitoring as a side effect. When the injection says "suppress forward momentum bias," the model must pause and evaluate whether it has committed to an early hypothesis. That pause IS self-monitoring. This observation is central to the [Cognitive Scaffolding Thesis](/blog/cognitive-scaffolding-thesis): scaffolds function as persistent attention anchors, and emergent self-monitoring is a direct consequence of that persistence.

This is an emergent capability: the model monitors itself because the injection blocks the shortcuts that let it skip monitoring.

A parallel emergence appeared in ARC-AGI-3 testing. At step 15 of a 25-step spatial navigation game, the harnessed agent spontaneously switched from natural language to symbolic mathematical notation: defining formal variables, computing coordinates algebraically, reasoning about movement vectors. The injection's suppression signal (`start_end_only_thinking`) did not instruct math. It constrained a failure mode. The agent found its own solution to that constraint. Suppression signals are behavioral pressures. The model's adaptation is emergent. See [What Happened When an LLM Taught Itself Symbolic Math](/blog/arc-agi-3-emergent-behaviors).

### Domain-Agnostic Suppression

One of the six reasoning domains had 0% retrieval precision during external benchmarks. The wrong abilities were retrieved for every task in that domain. Yet performance still improved by +8.5 percentage points. Full analysis: [62% of Tasks Got the Wrong Domain. It Didn't Matter.](/blog/domain-agnostic-suppression)

This means suppression signals are universal failure-mode blockers, not domain-specific procedures. Signals like "do not stop at surface-level explanations" work on causal tasks, temporal tasks, and abstract tasks equally. The product does not require perfect retrieval to produce measurable improvement.

### The Correctness Paradox

On custom tasks where baseline accuracy was already high (2.6 out of 3.0), correctness stayed flat while every quality dimension improved dramatically. This is the thesis-defining observation. Full data: [EjBench: 180 Professional Tasks](/blog/ejbench-180-tasks).

RA²R does not improve accuracy ceiling. It prevents reasoning decay. It changes HOW the agent arrives at its answer: more self-checking, more verification, more alternative consideration, more transparent reasoning chains. On published academic tasks with clearer right/wrong answers and lower baseline accuracy, correctness did improve (+0.14). The improvement exists when there is room for it.

---

## Limits

Suppression improves reasoning. It does not guarantee it.

- **Does not add domain knowledge.** If your agent fails because it lacks information, use RAG. Ejentum improves how the agent reasons about information it already has.
- **Operates at the prompt level.** It does not modify model weights, activations, or fine-tuning. It structures context in ways that measurably shift output quality.
- **Retrieval precision depends on query quality.** Short or highly ambiguous queries may not surface the optimal ability. Send the full task description, not a one-word summary.
- **Suppression is not absolute.** LLMs are probabilistic systems. Suppression significantly reduces failure rates. it does not guarantee zero failures.

---

## The Meta-Dimension

These six failure modes are not independent. Metacognitive failure amplifies every other failure mode. An agent that cannot flag its own uncertainty will confidently output reversed causes, confabulated timelines, and collapsed categories. Metacognition is the meta-dimension: it monitors the integrity of all other reasoning processes.

This is why Metacognitive abilities appear in the `fallback` synergy position of many ability chains. they act as a safety net, activating if the primary reasoning chain shows signs of drift.

For the full research paper covering these findings: [Under Pressure: Our First Research Paper](/blog/under-pressure-research-paper). For terminology definitions: [Glossary](/docs/glossary).
