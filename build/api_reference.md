# API Reference

Complete technical reference for the Ejentum Logic API v1. If you have not read the [Quickstart](/docs/quickstart), start there. This page is the full specification for builders integrating into production.

## Base URL

```
https://ejentum-main-ab125c3.zuplo.app
```

## Authentication

All requests require an API key passed as a Bearer token:

```
Authorization: Bearer YOUR_API_KEY
```

## Endpoints

### `POST /logicv1/`

Evaluate a natural language query and retrieve the optimal cognitive ability from the matched product layer.

#### Request

| Header | Value |
|:-------|:------|
| `Authorization` | `Bearer YOUR_API_KEY` |
| `Content-Type` | `application/json` |

#### Request Body

```json
{
  "query": "string",
  "mode": "reasoning"
}
```

| Field | Type | Required | Description |
|:------|:-----|:---------|:------------|
| `query` | String | Yes | The agent's task description or reasoning context. Send the full task, not a summary. Query specificity drives routing precision. |
| `mode` | String | No | Injection mode. Defaults to `reasoning`. See Modes table below. |

#### Modes

| Mode | Description | Plan | Response Key |
|:-----|:------------|:-----|:-------------|
| `reasoning` | Reasoning Harness. 311 abilities across 6 domains. | Ki | `reasoning` |
| `reasoning-multi` | Primary + cross-domain suppression graph with dynamic meta-checkpoint. | Haki | `reasoning-multi` |
| `anti-deception` | Blocks sycophancy, hallucination, prompt injection. 139 abilities. | Ki | `anti-deception` |
| `code` | Code generation, refactoring, architecture. 128 abilities. | Ki | `code` |
| `code-multi` | Primary + cross-domain engineering guards. | Haki | `code-multi` |
| `memory` | Perception sharpening, behavioral calibration. 101 abilities. | Ki | `memory` |
| `memory-multi` | Primary + cross-domain perceptual guards. | Haki | `memory-multi` |

#### Response: 200 OK

The response is a JSON array with a single object. The response key matches the mode name:

```json
[
  {
    "reasoning": "[NEGATIVE GATE]\nTreats the first visible symptom as the root cause...\n\n[PROCEDURE]\nStep 1: Enumerate all competing explanations...\n\n[REASONING TOPOLOGY]\nS1:enumerate -> S2:count_assumptions -> G1{covers_all?} --yes-> OUT --no-> S2[LOOP]\n\n[TARGET PATTERN]\nIteratively applies n-Whys deconstruction...\n\n[FALSIFICATION TEST]\nIf the proposed fix addresses a surface symptom...\n\nAmplify: depth first root search\nSuppress: symptom treatment bias; surface level stop"
  }
]
```

#### Response Keys

The response key always matches the mode you sent. Parse the value — it's a pre-rendered injection string ready to use.

The injection string is ready to use. Inject directly into your LLM's system message. No field assembly required. See [Integration Guides](/docs/integrations) for framework-specific patterns, or the [Agent Tool Guide](/docs/agent_tool_guide) for the full injection protocol.

## Dimensional Routing

The query is automatically matched against the abilities within your chosen mode. Each product layer has its own cognitive dimensions. The API returns the highest-scoring ability.

| Dimension | Activates On | Prevents |
|:-------|:-------------|:---------|
| Causal | why, cause, failure, root, reason, error | Correlation-causation confusion, causal reversal |
| Temporal | when, timeline, trend, predict, history, drift | Temporal hallucination, sequence loss |
| Spatial | where, near, position, layout, distance, boundary | Physical impossibility, topology violation |
| Simulation | what if, scenario, model, stress, test, network | Single-step myopia, feedback loop blindness |
| Abstract | meaning, concept, principle, analogy, metaphor | Category conflation, ontological errors |
| Metacognitive | think, reflect, bias, monitor, hallucination, fallacy | Hallucination spirals, infinite regression |

## Dimension Examples

Each query is automatically routed to the best-matching ability. The examples below show the reasoning operation activated per dimension.

| Dimension | Example Query | What the Scaffold Does |
|:----------|:-------------|:----------------------|
| Causal | "Why did our model accuracy degrade after retraining?" | Traces root cause. Suppresses symptom-treatment bias. |
| Temporal | "When will this migration complete based on current velocity?" | Estimates realistic duration. Suppresses optimism bias. |
| Spatial | "Are there overlapping boundaries in this network topology?" | Validates structural constraints. Suppresses boundary violations. |
| Simulation | "What if we remove the rate limiter from the payment service?" | Models downstream consequences. Suppresses first-order-only thinking. |
| Abstract | "What is the core principle behind this system's architecture?" | Distills core essence. Suppresses surface-level categorization. |
| Metacognitive | "My agent keeps contradicting itself across multi-step plans" | Audits reasoning coherence. Suppresses hallucination spirals. |

The response for any of these is a pre-rendered injection string in the mode you requested.

## Multi Modes

Multi modes (`reasoning-multi`, `code-multi`, `memory-multi`) retrieve a primary ability plus 3 cross-domain synergy abilities. Instead of injecting all 4 fully, multi mode extracts the architectural elements that generalize:

| Section | What it contains |
|:--------|:----------------|
| PRIMARY | Full ability: procedure, topology, falsification test |
| SUPPRESSION GRAPH | N-nodes extracted from all 4 topologies. Cross-domain failure guards. |
| META-CHECKPOINT | Dynamic self-check derived from the N-nodes. "Verify you did NOT: [list]" |
| ON_FAILURE | Escape pattern: ABANDON_GRAPH -> FREEFORM -> RE-ENTER |
| Amplify / Suppress | Merged signals from all 4 abilities |

The primary gives depth on the specific task. The suppression graph gives breadth across failure modes. The meta-checkpoint forces self-evaluation before output.

## Writing Better Queries

Query quality directly impacts retrieval precision. The hybrid search engine matches your query both semantically and lexically.

| Instead of... | Send... | Why |
|:-------------|:--------|:----|
| "Help me analyze this" | "Identify why customer churn increased 30% in Q3 after the pricing change" | Activates Causal + Temporal signals |
| "Fix my agent" | "My agent hallucinates causal chains, it says A causes B without tracing the mechanism" | Directly activates Causal suppression |
| "Think about this problem" | "Model the downstream consequences of removing the rate limiter from the payment service" | Activates Simulation + Causal |
| "What's going on?" | "Detect temporal drift in our prediction accuracy over the last 6 months" | Activates Temporal with high confidence |

**Key rule:** Send the agent's actual task description, not a meta-description of what you want. The router scores the semantic content, not the intent.

## Error Responses

| Status | Description |
|:-------|:------------|
| `400 Bad Request` | Missing or empty `query` field |
| `401 Unauthorized` | Missing or invalid API key |
| `429 Too Many Requests` | Rate limit exceeded |
| `500 Internal Server Error` | Backend processing failure |

### Error Body

```json
{
  "error": "string. description of the error"
}
```

## Rate Limits

| Limit | Value |
|:------|:------|
| Requests per minute | 100 per API key |
| Monthly calls (Free) | 100 total |
| Monthly calls (Ki) | 5,000 |
| Monthly calls (Haki) | 10,000 |

When rate-limited, the API returns `429` with a clear message and upgrade link. See [Pricing](/pricing) for plan details.

## Platform

The API runs on a global edge network with:

- API key validation and tier enforcement
- Per-key rate limiting
- DDoS protection
- Request logging and analytics

No internal infrastructure is directly accessible. The only public surface is the `/logicv1/` endpoint.

## API Versioning

The current endpoint is `/logicv1/`. The `v1` in the path is the API version.

- **Stability guarantee:** v1 response schemas will not change without a new version path.
- **Additions:** New fields may be added to the response (additive changes only).
- **Deprecation:** v1 will remain active for a minimum of 12 months after v2 is released.
