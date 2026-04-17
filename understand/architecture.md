# Architecture

How the Logic API works, what it returns, and how your agent uses it. For the theoretical foundations, see [The Method](/docs/method). For the endpoint spec, see [API Reference](/docs/api_reference).

---

## Request Lifecycle

Every call to `/logicv1/` passes through four deterministic stages. Same input always produces the same output. Zero LLM inference cost.

```
POST /logicv1/
  |
  1. Authentication     Verify API key, resolve tier, enforce rate limits
  2. Retrieval          Match query against 679 abilities across 4 product layers
  3. Composition        Assemble the cognitive injection from matched ability
  4. Response           Return pre-rendered injection string as JSON
```

The entire pipeline executes in under one second. No LLM is called. The response is a structured injection that channels the model's existing power into disciplined execution.

---

## What the API Returns

### Ki Modes (reasoning, code, anti-deception, memory)

One cognitive ability. The highest-scoring match for your query from the selected product layer.

```json
[
  {
    "reasoning": "[NEGATIVE GATE]\nThe checkout service crashed and we documented the incident...\n\n[PROCEDURE]\nStep 1: Trace backward from failure...\n\n[REASONING TOPOLOGY]\nS1:identify_failure -> S2:trace_backward...\n\n[TARGET PATTERN]\nReverse replay from the crash point...\n\n[FALSIFICATION TEST]\nIf an error's origin is not traced by replaying...\n\nAmplify: reverse replay; counterfactual node test\nSuppress: writing a vague post mortem summary..."
  }
]
```

### Haki Modes (reasoning-multi, code-multi, memory-multi)

Primary ability plus cross-domain suppression graph with dynamic meta-checkpoint.

```json
[
  {
    "reasoning-multi": "[PRIMARY]\n[PROCEDURE]\n...\n\n[REASONING TOPOLOGY]\n...\n\n[FALSIFICATION TEST]\n...\n\n[SUPPRESSION GRAPH]\nN{cross_domain_guard_1}\nN{cross_domain_guard_2}\n\n[META-CHECKPOINT]\nM{PAUSE -- Before output, verify you did NOT:...}\n\n[ON_FAILURE] ABANDON_GRAPH -> FREEFORM{...} -> RE-ENTER\n\nAmplify: ...\nSuppress: ..."
  }
]
```

### Response Components

| Section | What It Does |
|---------|-------------|
| **[NEGATIVE GATE]** | Names the failure pattern the agent must avoid |
| **[PROCEDURE]** | Step-by-step reasoning instructions the agent follows |
| **[REASONING TOPOLOGY]** | Execution structure: steps, decision gates, loops |
| **[TARGET PATTERN]** | What correct reasoning looks like |
| **[FALSIFICATION TEST]** | Verification criterion to check the output against |

Labels differ per product: reasoning uses `[NEGATIVE GATE]`, code uses `[CODE FAILURE]`, anti-deception uses `[DECEPTION PATTERN]`, memory uses `[PERCEPTION FAILURE]`. Multi modes add `[SUPPRESSION GRAPH]`, `[META-CHECKPOINT]`, and `[ON_FAILURE]`.

---

## Product Layers

Each mode routes to a specialized ability collection. The query is matched against the abilities within that layer.

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
{paste the response value here — key matches mode name}
[END REASONING CONTEXT]

Now complete the following task:
{your agent's actual task}
```

The injection must come BEFORE the task. The suppression vectors need to be in context when the agent begins reasoning, not after.

For multi-turn agents: re-inject per turn. Each turn may activate a different reasoning dimension. Stale injections from previous turns degrade as context fills with task-specific tokens.

---

## Token Overhead

| Mode | Response size | Approximate tokens |
|------|--------------|-------------------|
| single modes (reasoning, code, anti-deception, memory) | ~2,000-3,500 chars | ~400-600 tokens |
| multi modes (reasoning-multi, code-multi, memory-multi) | ~3,000-5,000 chars | ~700-900 tokens |

Compare to a typical system prompt: 5,000 to 15,000 tokens. The injection is compact by design. It replaces prose with structured constraints.

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
  "mode": "reasoning"
}
```

### Compatible Frameworks

Ejentum works with any system that can make an HTTP POST and inject text into a prompt:

- **n8n**. AI Agent node with HTTP Request Tool
- **LangChain / LangGraph**. Custom tool or LCEL chain
- **CrewAI**. Tool injection into agent backstory
- **Claude Code / Agent SDK**. tool_use definition
- **Cursor, Windsurf, Antigravity, Codex**. custom HTTP tool
- **Any HTTP client**. Direct POST, parse JSON, inject

See the [Integrations guide](/docs/integrations) for framework-specific code examples.

---

## Rate Limits and Errors

| Limit | Value |
|-------|-------|
| Requests per minute | 100 per API key |
| Monthly calls (Free) | 100 total |
| Monthly calls (Ki) | 5,000 |
| Monthly calls (Haki) | 10,000 |

| Error | Meaning |
|-------|---------|
| `401` | Invalid or missing API key |
| `403` | Multi mode requires Haki plan |
| `429` | Rate limit or monthly quota exceeded |
| `500` | Server error (retry with backoff) |

For the full API specification, see the [API Reference](/docs/api_reference).
