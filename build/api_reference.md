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

Evaluate a natural language query across six reasoning dimensions and retrieve the optimal reasoning payload.

#### Request

| Header | Value |
|:-------|:------|
| `Authorization` | `Bearer YOUR_API_KEY` |
| `Content-Type` | `application/json` |

#### Request Body

```json
{
  "query": "string",
  "mode": "single"
}
```

| Field | Type | Required | Description |
|:------|:-----|:---------|:------------|
| `query` | String | Yes | The agent's task description or reasoning context. Send the full task, not a summary. Query specificity drives routing precision. |
| `mode` | String | No | Injection mode: `single` (default, recommended) or `multi` (premium, compound suppression). Defaults to `single`. |

#### Modes

| Mode | Description | Plan | Response Key |
|:-----|:------------|:-----|:-------------|
| `single` | 1 ability, pure signal injection. Best for correctness. | Ki (starter) | `single_ability` |
| `multi` | 4 abilities, compound suppression via merged vectors. | Haki (premium) | `multi_ability` |

#### Response: 200 OK

The response is a JSON array with a single object. The key is `{mode}_ability` and the value is a pre-rendered injection string:

```json
[
  {
    "single_ability": "[NEGATIVE GATE]\nTreats the first visible symptom as the root cause...\n\nStep 1: Enumerate all competing explanations... Step 2: For each, count assumptions...\n\n[REASONING TOPOLOGY]\nS1:enumerate → S2:count_assumptions → G1{covers_all?} --yes→ OUT:enforced_default --no→ S3:add_minimum → S2[LOOP]\n\nAmplify: depth first root search; n whys traversal\nSuppress: symptom treatment bias; surface level stop\n\n[TARGET PATTERN]\nIteratively applies n-Whys deconstruction...\n\n[FALSIFICATION TEST]\nIf the proposed fix addresses a surface symptom..."
  }
]
```

#### Response Fields

| Field | Type | Description |
|:------|:-----|:------------|
| `single_ability` | String | Pre-rendered single-ability injection string |
| `multi_ability` | String | Pre-rendered multi-ability injection with merged vectors |

The injection string is ready to use — inject directly into your LLM's system message. No field assembly required.

## Dimensional Routing

The query is automatically scored across all six reasoning dimensions. The API returns the ability from the highest-scoring dimension.

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

## Multi-Ability Mode

The `multi` mode retrieves 4 abilities (1 primary + 3 synergies) and composes them into a single injection:

| Node | Role | Description |
|:-----|:-----|:------------|
| PRIMARY | Seed ability | The best-matching ability for your query, rendered in full |
| DEPENDENCY | Required prerequisite | An ability the primary requires — executed first |
| AMPLIFIER | Enhancement | An ability that strengthens the primary's reasoning |
| ALTERNATIVE | Fallback framework | A different analytical lens that challenges the primary's conclusions |

Multi-ability modes also include `[MERGED VECTORS]` (deduplicated amplification/suppression across all 4 abilities) and domain coverage scores.

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
  "error": "string — description of the error"
}
```

## Rate Limits

| Limit | Value |
|:------|:------|
| Requests per minute | 100 per API key |
| Monthly calls (Free) | 100 total |
| Monthly calls (Ki) | 10,000 |
| Monthly calls (Haki) | 50,000 |

When rate-limited, the API returns `429` with a clear message and upgrade link.

## Infrastructure

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
