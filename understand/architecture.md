# Architecture

How the Logic API works, what it returns, and how your agent uses it.

---

## Request Lifecycle

Every call to `/logicv1/` passes through four deterministic stages. Same input always produces the same output. Zero LLM inference cost.

```
POST /logicv1/
  |
  1. Authentication     Verify API key, resolve tier, enforce rate limits
  2. Retrieval          Match query against 311 abilities across 6 dimensions
  3. Composition        Assemble the cognitive scaffold from matched ability
  4. Response           Return pre-rendered injection string as JSON
```

The entire pipeline executes in under one second. No LLM is called. The response is a structured reasoning scaffold, not a generated output.

---

## What the API Returns

### Single Mode — Ki (single ability)

One cognitive scaffold. The highest-scoring ability for your query.

```json
[
  {
    "single_ability": "[REASONING INJECTION]\nPhase: MONITORING_LOOP | Confidence: ASSERT...\n\n[NEGATIVE GATE]\nThe checkout service crashed and we documented the incident...\n\n[REASONING TOPOLOGY]\nS1:identify_failure → S2:trace_backward...\n\n[TARGET PATTERN]\nReverse replay from the crash point...\n\n[COGNITIVE PAYLOAD]\nAmplify: reverse replay; counterfactual node test\nSuppress: writing a vague post mortem summary...\n\n[FALSIFICATION TEST]\nIf an error's origin is not traced by replaying..."
  }
]
```

### Multi Mode — Haki (multi ability)

Four synergized abilities composed into one compound scaffold. Includes merged suppression and amplification vectors.

```json
[
  {
    "multi_ability": "[PRIMARY]\nPhase: MONITORING_LOOP...\n\n[DEPENDENCY]\n...\n\n[AMPLIFIER]\n...\n\n[ALTERNATIVE]\n...\n\n[MERGED VECTORS]\nAmplify: reverse replay; counterfactual node test...\nSuppress: writing a vague post mortem summary..."
  }
]
```

### Response Components

| Section | What It Does |
|---------|-------------|
| **[NEGATIVE GATE]** | Names the failure pattern the agent must avoid |
| **[REASONING TOPOLOGY]** | Execution structure: steps, decision gates, loops |
| **[TARGET PATTERN]** | What correct reasoning looks like |
| **[COGNITIVE PAYLOAD]** | Suppression vectors, amplification vectors, cognitive style |
| **[FALSIFICATION TEST]** | Verification criterion to check the output against |

In multi mode, these appear for the primary ability plus three synergy abilities (dependency, amplifier, alternative), followed by merged vectors.

---

## The Six Reasoning Dimensions

Every query is scored across all six dimensions simultaneously. The highest-scoring dimension determines which ability is returned.

| Dimension | What It Addresses |
|-----------|------------------|
| Causal | Direction-of-causation errors, correlation treated as cause |
| Temporal | Sequence errors, confabulated timelines, duration bias |
| Spatial | Topology failures, boundary violations, structural constraints |
| Simulation | Counterfactual collapse, failure to model consequences |
| Abstraction | Category errors, over-generalization, metaphor/mechanism confusion |
| Metacognition | Hallucination spirals, reasoning drift, bias blindness |

A supply chain risk query scores high on Causal and Temporal. A market expansion question scores on Spatial, Simulation, and Abstraction. The routing is probabilistic, not categorical.

---

## How to Inject

Wrap the API response in delimiters and prepend to your agent's system message, BEFORE the task prompt:

```
[REASONING CONTEXT]
{paste the single_ability or multi_ability value here}
[END REASONING CONTEXT]

Now complete the following task:
{your agent's actual task}
```

The scaffold must come BEFORE the task. The suppression vectors need to be in context when the agent begins reasoning, not after.

For multi-turn agents: re-inject per turn. Each turn may activate a different reasoning dimension. Stale scaffolds from previous turns degrade as context fills with task-specific tokens.

---

## Token Overhead

| Mode | Response size | Approximate tokens |
|------|--------------|-------------------|
| Single (Ki) | ~2,500 characters | ~500 tokens |
| Multi (Haki) | ~4,500 characters | ~900 tokens |

Compare to a typical system prompt: 5,000 to 15,000 tokens. The scaffold is compact by design. It replaces prose with structured constraints.

---

## Integration

Your agent calls Ejentum like any other tool. One POST request, one JSON response, one injection.

### Request

```json
POST https://ejentum-main-ab125c3.zuplo.app/logicv1/
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "query": "Why did our deployment fail after the config change?",
  "mode": "single"
}
```

### Compatible Frameworks

Ejentum works with any system that can make an HTTP POST and inject text into a prompt:

- **n8n** — AI Agent node with HTTP Request Tool
- **LangChain / LangGraph** — Custom tool or LCEL chain
- **CrewAI** — Tool injection into agent backstory
- **Claude Code / Agent SDK** — tool_use definition
- **Cursor, Windsurf, Antigravity, Codex** — custom HTTP tool
- **Any HTTP client** — Direct POST, parse JSON, inject

See the [Integrations guide](../build/integrations.md) for framework-specific code examples.

---

## Rate Limits and Errors

| Limit | Value |
|-------|-------|
| Requests per minute | 100 per API key |
| Monthly calls (Free) | 100 total |
| Monthly calls (Ki) | 10,000 |
| Monthly calls (Haki) | 50,000 |

| Error | Meaning |
|-------|---------|
| `401` | Invalid or missing API key |
| `403` | Multi mode requires Haki plan |
| `429` | Rate limit or monthly quota exceeded |
| `500` | Server error (retry with backoff) |

For the full API specification, see the [API Reference](../build/api_reference.md).
