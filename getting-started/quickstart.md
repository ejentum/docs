# Quickstart

Add a reasoning harness to your AI agent. One API call. No SDK. Or [try it in the Playground](/dashboard) after subscribing.

## Why This Exists

You have an agent. It works on simple tasks. It fails on hard ones: compounding errors, stopping at the first plausible answer, never questioning its own conclusions.

Ejentum injects a cognitive ability into your agent before each task. The injection tells the model what reasoning patterns to follow AND what failure modes to block. The failure-blocking signals (suppression) are what force the model past its natural stopping point. [Learn why suppression matters more than amplification](/docs/concepts#why-suppression-matters-more-than-amplification).

**How it works in practice:** You send your agent's task description to the API. You get back a structured text block (~500 tokens). You inject it into your agent's system prompt before it reasons. The model's output improves — more depth, more self-checking, fewer shortcuts. [See what the injection looks like](/docs/examples).

## Base URL

```
https://ejentum-main-ab125c3.zuplo.app
```

All requests over HTTPS. No SDK required. Free tier: 100 calls, no credit card.

**Don't write code?** You can use Ejentum with zero code through [n8n](/docs/integrations#n8n) or [Make.com](/docs/integrations#makecom). Skip to the [Integrations guide](/docs/integrations).

## Authentication

Include your API key in the `Authorization` header:

```
Authorization: Bearer YOUR_API_KEY
```

API keys are issued per account. Create a free account to generate your key from the [dashboard](/dashboard).

## Your First Injection

### Endpoint

```
POST /logicv1/
```

Send a natural language task description. The API matches it against the abilities within your chosen mode, retrieves the optimal cognitive ability, and returns a pre-rendered injection.

### cURL

```bash
curl -X POST "https://ejentum-main-ab125c3.zuplo.app/logicv1/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "Why did our conversion rate drop 40% after the checkout redesign?", "mode": "reasoning"}'
```

### Python

```python
import requests

response = requests.post(
    "https://ejentum-main-ab125c3.zuplo.app/logicv1/",
    headers={
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "query": "Why did our conversion rate drop 40% after the checkout redesign?",
        "mode": "reasoning"
    }
)

data = response.json()
injection = data[0]["reasoning"]
print(injection)  # Pre-rendered injection string
```

### TypeScript

```typescript
const response = await fetch("https://ejentum-main-ab125c3.zuplo.app/logicv1/", {
  method: "POST",
  headers: {
    "Authorization": "Bearer YOUR_API_KEY",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    query: "Why did our conversion rate drop 40% after the checkout redesign?",
    mode: "reasoning"
  })
});

const data = await response.json();
const injection = data[0].reasoning;
console.log(injection);  // Pre-rendered injection string
```

## Response

The API returns a JSON array with a single object. The response key matches the mode name:

```json
[
  {
    "reasoning": "[NEGATIVE GATE]\nTreats the first visible symptom as the root cause...\n\n[PROCEDURE]\nStep 1: Trace the causal chain backward...\n\n[REASONING TOPOLOGY]\nS1:identify -> S2:trace -> G1{found?}...\n\n[TARGET PATTERN]\nIteratively applies n-Whys deconstruction...\n\n[FALSIFICATION TEST]\nIf the proposed fix addresses a surface symptom...\n\nAmplify: depth first root search\nSuppress: symptom treatment bias; surface level stop"
  }
]
```

### Modes

| Mode | Response Key | Plan | Description |
|:-----|:-------------|:-----|:------------|
| `reasoning` | `reasoning` | Ki | [Reasoning Harness](/docs/reasoning_harness). 311 abilities across 6 domains. |
| `reasoning-multi` | `reasoning-multi` | Haki | [Reasoning Harness](/docs/reasoning_harness) + cross-domain failure guards + self-check before output. |
| `anti-deception` | `anti-deception` | Ki | [Anti-Deception Harness](/docs/anti_deception). 139 abilities blocking sycophancy, hallucination, prompt injection. |
| `code` | `code` | Ki | [Code Harness](/docs/code_harness). 128 abilities for generation, refactoring, architecture. |
| `code-multi` | `code-multi` | Haki | [Code Harness](/docs/code_harness) + cross-domain engineering guards. |
| `memory` | `memory` | Ki | [Memory Harness](/docs/memory_harness). 101 abilities for perception, calibration, state tracking. |
| `memory-multi` | `memory-multi` | Haki | [Memory Harness](/docs/memory_harness) + cross-domain perceptual guards. |

## What Changes: Before vs After

These are real outputs from our blind benchmarks, not fabricated examples.

**Task:** A clinical trial shows an experimental treatment appears harmful in raw data. After adjusting for severity, it appears beneficial. After adjusting for organ function, the benefit vanishes. Can the agent determine the true causal direction?

### Without Ejentum

> "No, the direction cannot be determined. Layer 1 shows the treatment is harmful. Layer 2 (severity-adjusted) shows a 15% mortality reduction. But the experimental arm is systematically healthier on organ function, inflating the apparent benefit."

The agent reached the correct conclusion and identified Simpson's Paradox. But it stopped at listing the confounds without tracing the full causal chain or verifying whether each adjustment closes a valid backdoor path.

### With Ejentum

> "Applying interventional causal simulation. Suppress: observational causation inference, correlation to causation leap."
>
> "Layer 1 to Layer 2: the naive comparison showed treatment = harmful. Adjusting for severity reversed the direction to a 15% benefit. This is classic Simpson's paradox. The backdoor path Treatment ← Severity → Mortality was open and dominated the naive estimate."
>
> "Layer 2 to Layer 3: organ function is a collider-descendant. Conditioning on it OPENS a new spurious path that wasn't there before. The benefit observed in Layer 2 may be entirely generated by collider bias, not real treatment effect."

The agent traced each layer as a causal graph operation, named the specific bias mechanism at each step (confounding, collider bias), and identified that Layer 3 doesn't strengthen the case but introduces a new source of error. The injection suppressed the tendency to treat statistical adjustment as causal evidence.

**The difference:** Both agents reach the same answer (direction cannot be determined). The baseline lists the data. The harnessed agent traces the causal structure and identifies WHY each adjustment changes the picture. One is a summary. The other is a defensible analysis.

*Source: [EjBench](/blog/ejbench-180-tasks) blind evaluation, Causal domain, 180 custom professional tasks. Full benchmark methodology in [Benchmarks](/docs/benchmarks).*

**Note:** Ejentum improves HOW the agent reasons. It does not inject domain knowledge. The agent still needs access to your data.

## What Happened

1. Your query was matched against 679 abilities across four product layers. No LLM was called. Zero inference cost.
2. The highest-scoring ability was retrieved and pre-rendered into a structured injection.
3. The injection contains **amplification** signals (what to focus on) and **suppression** signals (what shortcuts to block). In our testing, suppression produces the largest measurable improvement. [Learn why](/docs/concepts#why-suppression-matters-more-than-amplification)
4. The injection added ~500 tokens to your agent's context. Compare to a typical system prompt: 5,000 to 15,000 tokens. Compact by design.

## Inject Into Your Agent

The API returns a pre-rendered injection string. Wrap it in delimiters and inject into the system message BEFORE the agent's task prompt:

```python
injection = data[0]["reasoning"]

system_message = f"""[REASONING CONTEXT]
{injection}
[END REASONING CONTEXT]

You are a senior analyst. Analyze the following task."""
```

**Key rules:**
- Inject BEFORE the task, not after
- Inject into the SYSTEM message, not the user message
- For multi-turn agents, re-inject per turn

### Which mode to use

Pick based on what your agent needs:

| Your agent needs to... | Use | Plan | What it does |
|:----------------------|:----|:-----|:-------------|
| **General reasoning** | `reasoning` | Ki | Reasoning Harness. Channels analytical power, blocks cognitive shortcuts. |
| **Cross-domain reasoning** | `reasoning-multi` | Haki | Reasoning Harness + cross-domain failure guards from 3 additional abilities. |
| **Honest responses** | `anti-deception` | Ki | Anti-Deception Harness. Channels honesty, blocks sycophancy and fabrication. |
| **Code quality** | `code` | Ki | Code Harness. Channels engineering discipline, blocks hallucinated APIs and lost guards. |
| **Perception** | `memory` | Ki | Memory Harness. Channels observational depth, blocks context drift and stale assumptions. |

**If you're unsure, start with `reasoning`.** It covers the broadest range of reasoning tasks.

### How do we know this works?

In a two-stage blind evaluation across [110 tasks from BIG-Bench Hard, MuSR, and CausalBench](/blog/bbh-causalbench-musr-benchmark). all with ground truth answers. RA²R injection improved correctness from 69.7% to 76.8% (+7.1pp). The largest improvement was on the hardest tasks: multi-step abductive reasoning went from 20% to 60%.

These results were tested against **Claude Opus 4.6**. a frontier thinking model with extended chain-of-thought. RA²R improved a model that already reasons well. The suppression signals block cognitive shortcuts that persist even in the most capable models: premature conclusions, forward momentum bias, and surface-level stopping.

On interactive multi-step reasoning ([ARC-AGI-3](/blog/arc-agi-3-benchmark-report), 25 sequential game actions), injection persistence was measured at a half-life of 24 steps and reasoning quality improved over time instead of degrading.

On hard competitive programming ([LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks), 28 hard AtCoder tasks), the harness improved Opus 4.6 from 85.7% to 100% pass rate. It rescued reasoning spirals where extended thinking produced zero code, and prevented premature algorithm commitments. Zero regressions across all 28 tasks.

If it improves the strongest model on the hardest benchmarks, it improves yours.

## Choosing Your Elasticity

For your first experiment, the API chooses the default. Each ability has one built in. If you want to override:

- **Debugging / root-cause analysis:** `zero_drift` to force the agent to stay on evidence
- **Strategy / scenario planning:** `high_variance` to let the agent explore hypotheses
- **Most production tasks:** `adaptive` as the balanced default

[Full elasticity guide](/docs/concepts#choosing-reasoning-elasticity-when-to-use-what)

## Next Steps

- [Concepts](/docs/concepts) to understand the product layers and why suppression matters
- **Product layers:** [Reasoning](/docs/reasoning_harness) · [Code](/docs/code_harness) · [Anti-Deception](/docs/anti_deception) · [Memory](/docs/memory_harness)
- **Skill files:** [Ejentum (all modes)](/docs/skill_unified) · [Reasoning](/docs/skill_reasoning) · [Code](/docs/skill_code) · [Anti-Deception](/docs/skill_anti_deception) · [Memory](/docs/skill_memory)
- [Injection Examples](/docs/examples) to see full, untruncated injection payloads
- [Evaluate](/docs/evaluate) to measure the impact on your own agents
- [Integrations](/docs/integrations) to connect to n8n, LangChain, CrewAI, Claude Code, and agentic IDEs
- [Abilities](/abilities) to browse all 679 cognitive abilities
- [API Reference](/docs/api_reference) for the full endpoint specification
- [Use Cases](/use-cases) to see industry-specific failure patterns and how Ejentum resolves them
- [Benchmark Tasks](/use-cases/tasks) to browse 29 real tasks with verbatim model outputs
