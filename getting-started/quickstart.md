# Quickstart

Add a reasoning scaffold to your AI agent. One API call. No SDK.

## Why This Exists

You have an agent. It works on simple tasks. It fails on hard ones: compounding errors, stopping at the first plausible answer, never questioning its own conclusions.

Ejentum adds a cognitive scaffold to your agent before each task. The scaffold tells the model what reasoning patterns to follow AND what failure modes to block. The failure-blocking signals (suppression) are what force the model past its natural stopping point. [Learn why suppression matters more than amplification](concepts.md#why-suppression-matters-more-than-amplification).

## Base URL

```
https://ejentum-main-ab125c3.zuplo.app
```

All requests over HTTPS. No SDK required. Free tier: 100 calls, no credit card.

**Don't write code?** You can use Ejentum with zero code through [n8n](../build/integrations.md#n8n) or [Make.com](../build/integrations.md#makecom). Skip to the [Integrations guide](../build/integrations.md).

## Authentication

Include your API key in the `Authorization` header:

```
Authorization: Bearer YOUR_API_KEY
```

API keys are issued per account. Create a free account to generate your key from the [dashboard](https://ejentum.com/dashboard).

## Your First Injection

### Endpoint

```
POST /logicv1/
```

Send a natural language task description. The API evaluates it across six reasoning dimensions, retrieves the optimal reasoning ability, and returns a structured injection payload.

### cURL

```bash
curl -X POST "https://ejentum-main-ab125c3.zuplo.app/logicv1/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "Why did our conversion rate drop 40% after the checkout redesign?", "mode": "single"}'
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
        "mode": "single"
    }
)

data = response.json()
injection = data[0]["single_ability"]
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
    mode: "single"
  })
});

const data = await response.json();
const injection = data[0].single_ability;
console.log(injection);  // Pre-rendered injection string
```

## Response

The API returns a JSON array with a single object. The key is `{mode}_ability` and the value is a pre-rendered injection string:

```json
[
  {
    "single_ability": "[NEGATIVE GATE — avoid this pattern]\nTreats the first visible symptom as the root cause...\n\nAmplify: depth first root search; n whys traversal\nSuppress: symptom treatment bias; surface level stop\nStyle: root cause isolation\n\n[TARGET PATTERN]\nIteratively applies n-Whys deconstruction...\n\n[FALSIFICATION TEST]\nIf the proposed fix addresses a surface symptom..."
  }
]
```

### Modes

| Mode | Response Key | Plan | Description |
|:-----|:-------------|:-----|:------------|
| `single` | `single_ability` | Ki (starter) | 1 ability, pure signal injection. Compact reasoning payload with amplification and suppression vectors. |
| `multi` | `multi_ability` | Haki (premium) | 4 abilities, compound suppression. Multi-ability chain with merged vectors across domains. |

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

The agent traced each layer as a causal graph operation, named the specific bias mechanism at each step (confounding, collider bias), and identified that Layer 3 doesn't strengthen the case but introduces a new source of error. The scaffold suppressed the tendency to treat statistical adjustment as causal evidence.

**The difference:** Both agents reach the same answer (direction cannot be determined). The baseline lists the data. The scaffolded agent traces the causal structure and identifies WHY each adjustment changes the picture. One is a summary. The other is a defensible analysis.

*Source: [EjBench](https://ejentum.com/blog/ejbench-180-tasks) blind evaluation, Causal domain, 180 custom professional tasks. Full benchmark methodology in [Benchmarks](../understand/benchmarks.md).*

**Note:** Ejentum improves HOW the agent reasons. It does not inject domain knowledge. The agent still needs access to your data.

## What Happened

1. Your query was matched against 311 reasoning abilities across six types of cognitive failure. No LLM was called. Zero inference cost.
2. The highest-scoring ability was retrieved and pre-rendered into a structured scaffold.
3. The scaffold contains **amplification** signals (what to focus on) and **suppression** signals (what shortcuts to block). In our testing, suppression produces the largest measurable improvement. [Learn why](concepts.md#why-suppression-matters-more-than-amplification)
4. The scaffold added ~500 tokens to your agent's context. Compare to a typical system prompt: 5,000 to 15,000 tokens. Compact by design.

## Inject Into Your Agent

The API returns a pre-rendered injection string. Wrap it in delimiters and inject into the system message BEFORE the agent's task prompt:

```python
injection = data[0]["single_ability"]

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
| **Get the right answer** | `single` | Ki (starter) | 1 ability, pure signal injection. Amplification and suppression vectors block cognitive shortcuts without constraining reasoning. |
| **Reason from multiple angles** | `multi` | Haki (premium) | 4 abilities with compound suppression. Covers more failure modes across domains with merged vectors. |

**If you're unsure, start with `single`.** It has the highest correctness improvement, the fewest regressions, and the smallest injection size.

### How do we know this works?

In a two-stage blind evaluation across [110 tasks from BIG-Bench Hard, MuSR, and CausalBench](https://ejentum.com/blog/bbh-causalbench-musr-benchmark) — all with ground truth answers — RA²R injection improved correctness from 69.7% to 76.8% (+7.1pp). The largest improvement was on the hardest tasks: multi-step abductive reasoning went from 20% to 60%.

These results were tested against **Claude Opus 4.6** — a frontier thinking model with extended chain-of-thought. RA²R improved a model that already reasons well. The suppression signals block cognitive shortcuts that persist even in the most capable models: premature conclusions, forward momentum bias, and surface-level stopping.

On interactive multi-step reasoning ([ARC-AGI-3](https://ejentum.com/blog/arc-agi-3-benchmark-report), 25 sequential game actions), scaffold persistence was measured at a half-life of 24 steps and reasoning quality improved over time instead of degrading.

If it improves the strongest model on the hardest benchmark, it improves yours.

## Choosing Your Elasticity

For your first experiment, the API chooses the default. Each ability has one built in. If you want to override:

- **Debugging / root-cause analysis:** `zero_drift` to force the agent to stay on evidence
- **Strategy / scenario planning:** `high_variance` to let the agent explore hypotheses
- **Most production tasks:** `adaptive` as the balanced default

[Full elasticity guide](concepts.md#choosing-reasoning-elasticity-when-to-use-what)

## Next Steps

- [Concepts](concepts.md) to understand the six dimensions and why suppression matters
- [Injection Examples](../build/examples.md) to see full, untruncated injection payloads
- [Evaluate](evaluate.md) to measure the impact on your own agents
- [Integrations](../build/integrations.md) to connect to n8n, LangChain, CrewAI, Claude Code, and agentic IDEs
- [Abilities](https://ejentum.com/abilities) to browse all 311 cognitive abilities
- [API Reference](../build/api_reference.md) for the full endpoint specification
- [Use Cases](https://ejentum.com/use-cases) to see industry-specific failure patterns and how Ejentum resolves them
- [Benchmark Tasks](https://ejentum.com/use-cases/tasks) to browse 29 real tasks with verbatim model outputs
