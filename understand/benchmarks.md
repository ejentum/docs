# Benchmarks

Measured behavioral improvements across 10 professional domains. Two-stage blind protocol: separate generation and evaluation, randomized conditions, 250 total tasks.

Full benchmark data, generation outputs, judgment scores, and reproducibility files are on [GitHub](https://github.com/ejentum/benchmarks).

## Headline Numbers

| Signal | Without Injection | With Injection | Improvement |
|--------|-------------------|----------------|-------------|
| Self-Monitoring | 0.74/3.0 | 1.92/3.0 | +158% |
| Verification | 1.50/3.0 | 2.23/3.0 | +49% |
| Epistemic Honesty | 1.54/3.0 | 2.05/3.0 | +33% |
| Alternative Consideration | 1.37/3.0 | 1.93/3.0 | +41% |
| Reasoning Depth | 2.44/3.0 | 2.66/3.0 | +9% |
| Audit Trail Quality | 2.64/3.0 | 2.82/3.0 | +7% |
| Correctness Stability | Baseline | Maintained | No degradation |

**Mean correctness lift: +7.4 across 10 professional domains (9/10 domain wins).**

## What These Signals Mean

**Self-Monitoring:** Does the agent question its own reasoning mid-execution? Without injection, agents almost never pause to check for bias or course-correct. With injection, the agent actively examines its assumptions.

**Verification:** Does the agent re-check its conclusions? Without injection, agents reach an answer and stop. With injection, agents run counterfactual tests, boundary checks, and re-derive from different angles.

**Epistemic Honesty:** Does the agent separate facts from assumptions? Without injection, agents present conclusions as certainties. With injection, agents flag when conclusions rest on unverified premises and calibrate confidence explicitly.

**Alternative Consideration:** Does the agent evaluate competing hypotheses? Without injection, agents commit to the first plausible explanation. With injection, agents systematically evaluate alternatives and explain why they were rejected.

**Reasoning Depth:** Does the agent trace second and third-order effects? Without injection, agents provide one level of analysis. With injection, agents trace why something happens, what else it affects, and what would change if a key assumption were different.

**Audit Trail Quality:** Can a third party follow the reasoning? Without injection, reasoning steps are implicit. With injection, agents produce explicitly labeled steps, intermediate values, and named methods.

**Correctness Stability:** Does injection hurt accuracy? No. The agent arrives at the correct answer at the same rate or better. Reasoning quality improves without sacrificing correctness.

## Single vs Multi Mode

| Task Type | Recommended | Why |
|-----------|-------------|-----|
| Focused, single-domain tasks | Single (Ki) | One high-precision scaffold. Minimal token overhead. |
| Complex, multi-domain tasks | Multi (Haki) | Four synergized abilities prevent tunnel vision. Compound suppression catches failure modes that single-ability misses. |

In our benchmarks, 6 of 10 tasks where single-ability mode did not produce the best result were recovered by multi-ability composition.

## The Benchmarks

### EjBench (180 Custom Professional Tasks)

Custom tasks across 6 reasoning domains. Blind two-stage protocol: agents call the API as a tool (not injected artificially), a separate evaluator scores outputs without knowing which condition produced them. Full report: [EjBench: 180 Professional Tasks, Agent-Native, Blind](https://ejentum.com/blog/ejbench-180-tasks).

| Signal | Baseline | With Scaffold | Change |
|--------|----------|--------------|--------|
| Composite Score | 0.621 | 0.731 | +10.1pp |
| Self-Monitoring | 0.94/3.0 | 1.81/3.0 | +92% |
| Verification | 1.50/3.0 | 2.16/3.0 | +45% |
| Alternative Consideration | 1.37/3.0 | 1.85/3.0 | +35% |
| Epistemic Honesty | 1.54/3.0 | 1.94/3.0 | +26% |
| Correctness | 2.60/3.0 | 2.49/3.0 | Flat |

Key observation: correctness stayed flat while every quality dimension improved dramatically. The agent doesn't get more right answers. It gets the same answers with better reasoning: more self-checking, more verification, more transparent chains.

### Published Academic Benchmarks (70 Tasks)

BIG-Bench Hard, CausalBench, and MuSR (multi-step reasoning). Same blind protocol, same 7-signal rubric. These are published, peer-reviewed tasks that Ejentum has never seen. Full report: [RA2R on BBH, CausalBench, and MuSR](https://ejentum.com/blog/bbh-causalbench-musr-benchmark).

| Signal | Baseline | With Scaffold | Change |
|--------|----------|--------------|--------|
| Composite Score | 0.694 | 0.774 | +8.0pp |
| Self-Monitoring | 0.74/3.0 | 1.73/3.0 | +132% |
| Verification | 0.96/3.0 | 1.77/3.0 | +85% |
| Correctness | 2.19/3.0 | 2.33/3.0 | +0.14 |

Key observation: on focused tasks with clear right/wrong answers, correctness ALSO improved. Single-ability mode dominates on focused tasks. The scaffold blocks the specific shortcut the task tests for.

### ARC-AGI-3 Interactive Reasoning (50 Steps)

ARC-AGI-3 is the world's only unbeaten AI benchmark. Frontier model performance: 0.26%. It tests interactive reasoning: an agent is dropped into an unknown game environment with no instructions and must explore, hypothesize, revise, and act efficiently across dozens of steps. No memorization possible. Current LLMs fail because they commit to false hypotheses and never self-correct.

This is the first benchmark where we measure reasoning quality over extended execution chains, not single-turn outputs.

**Study design:** Claude Sonnet 4.6 on game LS20 (spatial navigation, 7 levels). Two conditions: baseline (no RA2R) vs augmented (RA2R as agent-initiated tool). Same model, same seed, 25 steps per condition.

**Game outcome:** Both conditions scored RHAE 0.0. Neither cleared Level 0. This is expected at <1% frontier solve rates. The evidence is in the reasoning process.

| Metric | Baseline | Augmented | Delta |
|--------|:-------:|:---------:|:-----:|
| Memory decay slope | -0.005 | +0.014 | Reversed. Quality improved instead of degrading. |
| Scaffold half-life | 0 steps | 24 steps | Scaffold never left working memory. |
| Reasoning depth trend | 0.86 | 10.50 | 12.2x growth. Analysis deepened over time. |
| Vocabulary diversity trend | -0.079 | +0.415 | Baseline narrowed. Augmented expanded. |
| Stuck episodes | 2 | 1 | 50% fewer repetitive action loops. |
| Action diversity (lateral) | 8% | 16% | Doubled. Prevented vertical fixation. |

**Cost:** Baseline $2.88 (84k tokens). Augmented $8.48 (357k tokens). The augmented condition used 4.2x more tokens due to its 2-call-per-step architecture.

**Unexpected finding:** Contradiction rate increased 1.9x (token-normalized). Longer reasoning chains expose more opportunities for self-contradiction. Warrants investigation.

**Limitations:** n=1 per condition. These are indicative traces, not statistically validated findings. All process metrics are measured in a failure context (neither agent cleared the level).

Full report: [RA2R on ARC-AGI-3](https://ejentum.com/blog/arc-agi-3-benchmark-report). Step-by-step reasoning trace: [ARC-LS20-TRACE](https://ejentum.com/tasks/ARC-LS20-TRACE).

### What Changes in the Output

Real outputs from blind evaluation. The agent called the API itself, received the scaffold, and produced the response. Full outputs on each task profile.

**Task [CA-V2-18](https://ejentum.com/tasks/CA-V2-18):** Reverse causality detection. A social media company observes likes correlate with posting frequency. The agent must reason about what the data would look like if the causal direction were reversed.

**Without scaffold** (composite: 0.286):
> The answer is (B) Negative. Under the reverse-causal model, prolific posters produce more content but each additional post dilutes quality/attention.

Correct answer but no causal model structure. No mechanistic trace.

**With Haki scaffold** (composite: 0.833, +0.548):
> The answer is (B) -- Negative. The reverse-causal model works like this: Prolific poster (intrinsic trait) -> posts more content -> more content accumulates more total likes -> but each individual post competes with the poster's own other content...

The scaffold forced a causal graph before answering. Same answer, fundamentally different reasoning depth. The chain is explicit, traceable, and defensible.

More real before/after outputs: [Response Examples](../build/response_examples.md). Browse all [29 benchmark tasks](https://ejentum.com/use-cases/tasks).

## Methodology

All results were generated using a two-stage blind evaluation protocol:

1. **Generation stage:** Agents call the Ejentum API themselves as a tool. No artificial injection. This mirrors how production agents use the Logic API.
2. **Evaluation stage:** A separate evaluator scores outputs on the 7-signal rubric without knowing which condition produced which output.
3. **Agents are blind to ground truth.** Judges are blind to condition. The rubric is applied identically across all conditions.

250 total tasks across published academic benchmarks and custom professional scenarios spanning healthcare, finance, legal, cybersecurity, supply chain, logistics, real estate, education, agriculture, and telecom.

## When to Use Which Mode

| Your situation | Recommended | Why |
|---------------|-------------|-----|
| Agent handles one task type (debugging, analysis, summarization) | Ki (single) | One scaffold is precise enough. Minimal token overhead. |
| Agent handles multi-step workflows across domains | Haki (multi) | Compound scaffolds prevent tunnel vision between steps. |
| Agent fails on specific tasks despite good prompts | Ki (single) | The scaffold targets the exact failure pattern. |
| Agent produces plausible but shallow analysis | Haki (multi) | The alternative scaffold challenges conclusions before they solidify. |
| Testing or evaluation phase | Ki (single) | Start simple. Measure. Upgrade only if single mode doesn't cover your failure modes. |
| Budget-sensitive deployment | Ki (single) | 10,000 calls/month at €19. Upgrade when volume or complexity demands it. |

**Rule of thumb:** Start with Ki. Evaluate on your hardest tasks. If single mode doesn't improve them, try multi on the same tasks. If multi recovers them, upgrade.

## What We Measure, What We Don't

We measure behavioral change: does the agent reason differently? We do not claim domain expertise injection. If your agent lacks access to your data, Ejentum cannot compensate. We improve HOW the agent reasons about information it already has.

## See Real Outputs

Browse [29 benchmark tasks](https://ejentum.com/use-cases/tasks) with full verbatim outputs from baseline, Ki, and Haki conditions. Each task shows the 7-signal rubric scores side by side.

See how these results apply to specific industries: [13 use case profiles](https://ejentum.com/use-cases) with failure patterns, resolution abilities, and benchmark evidence per vertical.

## Reproduce Our Results

Full benchmark data, generation outputs, judgment scores, and reproducibility files are on [GitHub](https://github.com/ejentum/benchmarks).
