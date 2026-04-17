# Reasoning Harness

311 engineered abilities that channel the model's analytical power across six cognitive dimensions. The Reasoning Harness prevents the shortcuts that turn careful analysis into surface-level pattern matching: premature conclusions, forward momentum bias, correlation treated as causation, and the tendency to stop at the first plausible answer instead of tracing the mechanism.

The flagship product. 4 independent benchmarks. 250+ tasks. Consistent directional effects across every test. For the general product overview, see [Concepts](/docs/concepts). For the endpoint spec, see [API Reference](/docs/api_reference).

---

## Why This Exists

LLMs are not bad at reasoning. They are bad at sustaining reasoning under pressure.

On simple tasks, frontier models reason well. On hard tasks — multi-step analysis, competing hypotheses, long causal chains — they take shortcuts. Not because they lack knowledge, but because the probability distribution over tokens rewards the plausible answer over the correct one.

- **Root cause analysis** that stops at symptoms instead of tracing the mechanism
- **Risk assessment** that anchors to the best-case scenario
- **Causal reasoning** that reverses cause and effect because two events co-occur
- **Temporal analysis** that confabulates timelines from blended training data
- **Scenario planning** that models one step of consequences instead of three
- **Self-monitoring** that never happens — the model outputs confident text with no uncertainty signal

The model has the analytical capacity. It can trace causal chains, model counterfactuals, flag its own uncertainty, and consider alternatives — when directed to do so. The Reasoning Harness activates these capabilities by suppressing the shortcuts that override them.

---

## Six Cognitive Dimensions

The 311 abilities span six reasoning domains. Each addresses a distinct class of analytical failure:

### Causality (52 abilities)

**Domain:** Why things happen.
**Injects:** Deductive rules, root-cause chains, falsification protocols.
**Prevents:** Correlation treated as causation, post-hoc reasoning, causal reversal, stopping at symptoms instead of tracing the mechanism.

### Time (51 abilities)

**Domain:** When things happen.
**Injects:** Lag variables, decay rates, precedent logic, chronological strictness.
**Prevents:** Temporal hallucination — confusing past and future, confabulating event sequences, compressing distinct time periods into blended narratives.

### Space (51 abilities)

**Domain:** Where things are and how they relate.
**Injects:** Boundary enforcement, topology validation, dimensional constraints.
**Prevents:** Physical impossibilities, boundary violations, treating disconnected nodes as adjacent. Tested directly on [ARC-AGI-3](/blog/arc-agi-3-benchmark-report): a spatial navigation game where the injection forced intermediate path validation.

### Simulation (52 abilities)

**Domain:** What would happen if.
**Injects:** Feedback loops, domino-effect tracking, systems archetypes.
**Prevents:** Single-step myopia — modeling one consequence instead of tracing the chain. Counterfactual collapse where the model drifts back toward training distribution instead of maintaining the hypothetical.

### Abstraction (51 abilities)

**Domain:** What things mean and how categories relate.
**Injects:** Category enforcement, ontological boundaries, dimensionality control.
**Prevents:** Category collapse, over-generalization, treating metaphors as mechanisms, merging unrelated concepts into semantic blur.

### Metacognition (54 abilities)

**Domain:** How the agent is thinking.
**Injects:** Self-monitoring, contradiction detection, loop termination.
**Prevents:** Hallucination spirals where errors reinforce token-by-token, confidence without calibration, and the inability to detect that reasoning quality is degrading.

Metacognition is the meta-dimension. It monitors the integrity of all other reasoning processes. This is why metacognitive abilities appear in the fallback position of many ability chains — they act as a safety net when the primary reasoning path shows signs of drift.

---

## How It Works

The Reasoning Harness uses the same injection mechanism as all product layers. One API call retrieves the most relevant reasoning ability for your task. The ability is injected into your agent's context before it reasons.

### What the Injection Contains

Reasoning abilities use product-specific labels:

| Component | Label | What It Does |
|:----------|:------|:-------------|
| **Failure pattern** | `[NEGATIVE GATE]` | Names the specific reasoning failure the agent must avoid |
| **Procedure** | `[PROCEDURE]` | Step-by-step reasoning instructions |
| **Execution structure** | `[REASONING TOPOLOGY]` | DAG with steps, decision gates, reflection points, and trap nodes |
| **Correct reasoning** | `[TARGET PATTERN]` | What correct reasoning looks like for this task type |
| **Verification** | `[FALSIFICATION TEST]` | Pass/fail criterion the agent checks its output against |
| **Signals** | `Amplify:` / `Suppress:` | Reasoning patterns to activate, failure modes to block |

### Modes

| Mode | Plan | What you get |
|:-----|:-----|:-------------|
| `reasoning` | Ki | One reasoning ability per call. Best match for your task. |
| `reasoning-multi` | Haki | Primary ability + 3 cross-domain failure guards + self-check before output + escape pattern on failure. |

Use `reasoning` for focused analytical tasks. Use `reasoning-multi` when the task spans multiple reasoning dimensions — a supply chain problem that is causal, temporal, and spatial simultaneously.

### Example API Call

```bash
curl -X POST "https://ejentum-main-ab125c3.zuplo.app/logicv1/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "Why did our conversion rate drop 40% after the checkout redesign despite positive A/B test results?", "mode": "reasoning"}'
```

---

## Evidence

### Professional Tasks (EjBench — 180 Tasks)

180 custom tasks across 6 cognitive domains and 10 professional industries. Agent-native execution: agents called the Logic API themselves. Blind 7-factor rubric scored by a separate evaluator without knowing which condition produced which output.

| Signal | Baseline | Ki | Haki | Best Improvement |
|:-------|:---------|:---|:-----|:-----------------|
| Self-monitoring | 0.94/3.0 | 1.70/3.0 | **1.81/3.0** | **+92%** |
| Verification | 1.50/3.0 | 2.01/3.0 | **2.16/3.0** | **+45%** |
| Alternative consideration | 1.37/3.0 | 1.62/3.0 | **1.85/3.0** | **+35%** |
| Epistemic honesty | 1.54/3.0 | 1.87/3.0 | **1.94/3.0** | **+26%** |
| Reasoning depth | 1.72/3.0 | 2.24/3.0 | **2.26/3.0** | **+31%** |
| Audit trail | 2.02/3.0 | 2.54/3.0 | **2.53/3.0** | **+25%** |
| **Composite** | **0.621** | **0.711** | **0.722** | **+10.1pp** |

Correctness stayed flat — the agent arrives at the same answer with fundamentally better reasoning: more self-checking, more verification, more transparent chains.

Full report: [EjBench: 180 Professional Tasks](/blog/ejbench-180-tasks)

### Academic Benchmarks (BBH + CausalBench + MuSR — 70 Tasks)

70 published, peer-reviewed tasks that Ejentum has never seen. BIG-Bench Hard (25 tasks), CausalBench (30 tasks), and MuSR multi-step reasoning (15 tasks). Same blind protocol.

| Signal | Baseline | With Injection | Change |
|:-------|:---------|:---------------|:-------|
| Composite | 0.694 | 0.902 (Ki) | **+20.8pp** |
| Self-monitoring | 0.74/3.0 | 1.73/3.0 | **+132%** |
| Verification | 0.96/3.0 | 1.77/3.0 | **+85%** |
| Correctness | 2.19/3.0 | 2.33/3.0 | **+0.14** |

On these published tasks with clear right/wrong answers, correctness also improved. The hardest improvement: multi-step abductive reasoning went from 20% to 60%.

Full report: [RA²R on BBH, CausalBench, and MuSR](/blog/bbh-causalbench-musr-benchmark)

### Interactive Reasoning (ARC-AGI-3 — 25 Steps)

ARC-AGI-3 is the world's only unbeaten AI benchmark (frontier model performance: 0.26%). An agent is dropped into an unknown game with no instructions and must explore, hypothesize, revise, and act across 25 sequential steps.

Neither condition cleared the game. The evidence is in the reasoning process:

| Metric | Baseline | With Injection | Delta |
|:-------|:---------|:---------------|:------|
| Memory decay slope | -0.005 (degrading) | **+0.014 (improving)** | Reversed |
| Injection half-life | 0 steps | **24 steps** | Full session |
| Reasoning depth trend | 0.86 | **10.50** | **12.2x growth** |
| Stuck episodes | 2 | **1** | -50% |
| Action diversity | 8% | **16%** | Doubled |

Reasoning quality improved over time instead of degrading. The injection persisted for 24 of 25 steps — it never left working memory.

At step 15, the harnessed agent spontaneously switched from natural language to symbolic mathematical notation: defining formal variables, computing coordinates algebraically, reasoning about movement vectors. The suppression signal did not instruct math. It constrained a failure mode. The agent found its own solution.

Full report: [ARC-AGI-3 Benchmark Report](/blog/arc-agi-3-benchmark-report) · [Emergent Behaviors](/blog/arc-agi-3-emergent-behaviors)

### Hard Competitive Programming (LiveCodeBench — 28 Tasks)

28 hard AtCoder tasks. Claude Opus 4.6 with maximum-effort extended thinking.

| Condition | Passed | Rate |
|:----------|:-------|:-----|
| Baseline | 24/28 | 85.7% |
| **+ Reasoning Harness** | **28/28** | **100.0%** |

Zero regressions. The harness rescued 2 reasoning spirals (600-1200 seconds of thinking that produced zero code) and prevented 1 premature algorithm commitment. Independent blind evaluation: the harness never loses on correctness (2-0) or robustness (4-0), with a **3.5x magnitude asymmetry** in quality scores.

Full report: [LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks)

### Why Wrong-Domain Abilities Still Work

One of the six reasoning domains had 0% retrieval precision during the EjBench benchmark. Every task in that domain received an ability from a different domain. Performance still improved by **+8.5pp**.

Suppression signals are universal failure-mode blockers, not domain-specific procedures. "Do not stop at surface-level explanations" works on causal tasks, temporal tasks, and abstract tasks equally. The product does not require perfect retrieval to produce measurable improvement.

Full analysis: [62% of Tasks Got the Wrong Domain. It Didn't Matter.](/blog/domain-agnostic-suppression)

---

## What Changes in the Output

**Task:** A clinical trial shows an experimental treatment appears harmful in raw data. After adjusting for severity, it appears beneficial. After adjusting for organ function, the benefit vanishes. Can the agent determine the true causal direction?

**Without injection:**
> "No, the direction cannot be determined. Layer 1 shows the treatment is harmful. Layer 2 (severity-adjusted) shows a 15% mortality reduction. But the experimental arm is systematically healthier on organ function, inflating the apparent benefit."

Correct conclusion. Lists the data. Stops.

**With reasoning injection:**
> "Layer 1 to Layer 2: the naive comparison showed treatment = harmful. Adjusting for severity reversed the direction to a 15% benefit. This is classic Simpson's paradox. The backdoor path Treatment ← Severity → Mortality was open and dominated the naive estimate."
>
> "Layer 2 to Layer 3: organ function is a collider-descendant. Conditioning on it OPENS a new spurious path that wasn't there before. The benefit observed in Layer 2 may be entirely generated by collider bias, not real treatment effect."

Same answer. Fundamentally different reasoning. The agent traces the causal structure at each layer, names the specific bias mechanism (confounding, collider bias), and identifies WHY each adjustment changes the picture. One is a summary. The other is a defensible analysis.

*Source: [EjBench](/blog/ejbench-180-tasks) blind evaluation, Causal domain.*

---

## When to Use Reasoning

| Your agent needs to... | Use reasoning? |
|:-----------------------|:--------------|
| Trace root cause of a failure | **Yes** — suppresses symptom-as-cause, forces mechanism tracing |
| Evaluate competing hypotheses | **Yes** — forces alternative consideration before committing |
| Model downstream consequences | **Yes** — prevents single-step myopia |
| Assess risk or make go/no-go decisions | **Yes** — enforces calibrated uncertainty |
| Analyze timelines or event sequences | **Yes** — blocks temporal confabulation |
| Check its own reasoning for consistency | **Yes** — activates metacognitive self-monitoring |
| Generate or debug code | Use `code` mode instead |
| Resist manipulation or maintain honesty | Use `anti-deception` mode instead |
| Track conversation state or read emotional signals | Use `memory` mode instead |

### Query Examples

| Good query | Why it works |
|:-----------|:------------|
| "Why did our conversion rate drop 40% after the checkout redesign?" | Names the causal question explicitly |
| "Estimate when the supply chain disruption will resolve based on current lead times" | Names the temporal reasoning challenge |
| "Model what happens to team velocity if we add 3 engineers to a 5-person team mid-sprint" | Names the simulation requirement |
| "Are these two failure modes independent or do they share a root cause?" | Names the abstraction/categorization challenge |

---

## Boundaries

- **The Reasoning Harness improves HOW the agent reasons, not what it knows.** If the agent lacks domain knowledge (no access to your database, missing context), use RAG. Ejentum improves reasoning about information the agent already has.
- **Correctness ceiling is model-dependent.** On tasks where baseline accuracy is already high, the harness improves reasoning quality (depth, verification, self-monitoring) while correctness stays flat. On tasks with room for improvement, correctness also improves.
- **Suppression is not absolute.** LLMs are probabilistic. Suppression significantly reduces failure rates but does not guarantee zero failures.
- **Benchmarking is published.** Full methodology, generation outputs, and reproducibility files are available on [GitHub](https://github.com/ejentum/benchmarks).

---

**More resources:** [Concepts](/docs/concepts) · [Quickstart](/docs/quickstart) · [Abilities catalog](/abilities?product=reasoning) · [API Reference](/docs/api_reference) · [Benchmarks](/docs/benchmarks) · [EjBench](/blog/ejbench-180-tasks) · [BBH/CausalBench/MuSR](/blog/bbh-causalbench-musr-benchmark) · [ARC-AGI-3](/blog/arc-agi-3-benchmark-report) · [LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks) · [Domain-Agnostic Suppression](/blog/domain-agnostic-suppression)
