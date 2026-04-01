# Agent Tool Guide

> **For the agent-facing skill file** (drop into IDE rules directories): see [Agent Skill File](/docs/agent_skill)
> This document is the developer-facing reference with code examples and architectural detail.

This guide teaches any AI agent — Claude, GPT, Gemini, Llama, or a custom framework — how to use the Ejentum RA²R (Reasoning Ability-Augmented Retrieval) Logic API as a tool.

**RA²R is not RAG.** RAG retrieves knowledge (documents, facts). RA²R retrieves reasoning abilities (cognitive scaffolds that change HOW you think, not what you know).

---

## Before You Start

You need:
- An API key (starts with `zpka_`)
- An LLM to augment
- A task to solve
- HTTP capability (any language, any framework)

---

## The 6-Gate Protocol

Each gate must pass before proceeding. Do not skip gates. Do not proceed with partial results.

### Gate 1: Authenticate

```
POST https://ejentum-main-ab125c3.zuplo.app/logicv1/
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

Gate check: Do you have a valid `zpka_` key? If no, register at the developer portal.

### Gate 2: Decompose Your Task (If Multi-Step)

If your task has multiple distinct reasoning challenges, break it into 1-5 discrete sub-problems. Each sub-task gets its own API call.

| Monolithic task (decompose) | Decomposed sub-tasks |
|:---------------------------|:----------------------------|
| "Fix the app" | "Trace root cause of the login failure" |
| | "Estimate impact on active user sessions" |
| | "Design the remediation with zero-downtime constraint" |

| Atomic task (send as-is) | Why no decomposition needed |
|:------------------------|:---------------------------|
| "Summarize this incident report" | Single reasoning mode — decomposing would fragment context |
| "Classify this support ticket" | Single operation — one ability covers it completely |

Different sub-tasks activate different reasoning domains automatically. Sending one giant multi-step query wastes the routing intelligence. But forcing decomposition on a genuinely atomic task wastes API calls and can degrade quality.

Gate check: Does your task require multiple distinct reasoning modes? If yes, decompose into 1-5 sub-tasks. If it's a single coherent operation, send it as one query.

### Gate 3: Send Request

```json
{
  "query": "your natural language task description",
  "mode": "single"
}
```

**Modes:**

| Mode | Best for | What it does |
|:-----|:---------|:-------------|
| `single` | Default. 1 ability, pure signal injection. Plan: Ki (starter). | Suppression + amplification on a single query. ~2,000 chars. |
| `multi` | Cross-dimensional reasoning. 4-ability synergy chain. Plan: Haki (premium). | Compound suppression across multiple dimensions. ~4,000 chars. |

Gate check: Valid JSON with `query` (string) and `mode` (string)? If no, fix.

### Gate 4: Validate Response

**Do not blindly inject the response.** Quarantine it first.

The API returns a JSON array with one object. The key is `{mode}_ability` and the value is a pre-rendered injection string:

```json
[{"single_ability": "[REASONING INJECTION]\n...\nAmplify: ...\nSuppress: ..."}]
```

**Quarantine checks:**

1. **Is it valid JSON and non-empty?** → if no, degrade gracefully (proceed without injection)
2. **Does the response key match your mode?** → `single_ability` or `multi_ability`. If the key is missing, the mode was invalid.
3. **Does the injection string contain expected markers?** → `[REASONING INJECTION]`, `[NEGATIVE GATE]`, `Amplify:`, `Suppress:`
4. **Is the ability relevant?** → read the `[NEGATIVE GATE]` section — does it describe a failure mode related to your task? If it describes an unrelated scenario, re-query with a more specific description.

**What a failed quarantine looks like:**
- You asked about database optimization, but the `[NEGATIVE GATE]` describes a marketing strategy failure → **irrelevant ability**. Re-query: "Optimize SQL query performance by analyzing execution plans and index usage" instead of "Make the database faster."

**Error responses (non-200):**
- `401` — invalid API key. Check the `Authorization: Bearer` header.
- `429` — rate limited. Wait and retry. The `Retry-After` header tells you when.
- `500` — server error. Degrade gracefully — proceed without injection.

Gate check: Response is valid JSON, key matches mode, markers present, ability is relevant? If no, re-query or proceed without.

### Gate 5: Inject Into Reasoning

**Rules (non-negotiable):**

1. Inject **BEFORE** the task instructions, not after
2. Inject into the **SYSTEM** message (or equivalent first-position context — see below)
3. Use **delimiters**: `[REASONING CONTEXT]...[END REASONING CONTEXT]`
4. In multi-turn conversations, **re-inject per turn**

**Template:**
```
[REASONING CONTEXT]
{the value of the {mode}_ability key from the API response}
[END REASONING CONTEXT]

{your actual task instructions here}
```

**Extraction:** The API returns `[{"single_ability": "<injection string>"}]` (or `multi_ability` for multi mode). The value of that key — the injection string — is what goes between the delimiters. Not the full JSON, just the string value.

**Frameworks without a system message slot:**
Some platforms (n8n, Make.com, some custom agents) don't have a separate system message. In these cases, prepend the injection to whatever text field the LLM reads first — the agent's `backstory`, `instructions`, or the first user message. The principle is positional: the injection must be the first structured content the model processes, regardless of what the field is called.

Gate check: Injection is first in system message, with delimiters, before task? If no, reposition.

### Gate 6: Execute and Verify

Run your task with the augmented reasoning. Then check:

- Does your output pass the `[FALSIFICATION TEST]` from the ability?
- If no → re-query the API with: `"Agent failed to {task}. Error: {error}. Retry with corrective reasoning."`
- This often triggers a Metacognitive ability that wasn't selected on the first pass

Gate check: Output passes falsification test? If no, retry (max 2 attempts).

---

## What You're Retrieving: The Cognitive Architecture

When you call the API, you're not getting a document or a prompt template. You're getting a **reasoning ability** — a structured cognitive scaffold drawn from a graph of 311 abilities across 6 reasoning domains. Understanding what you receive is essential to using it correctly.

### The 6 Reasoning Domains

Every ability belongs to one of 6 domains. The API routes your query to the best-matching ability across all domains simultaneously — you don't choose the domain, the hybrid search does.

| Domain | Code | What It Injects | What It Prevents | When It Activates |
|:-------|:-----|:---------------|:-----------------|:-----------------|
| **Causality** | CA | Root-cause chains, falsification protocols, deductive rules | Correlation-causation confusion, treating symptoms as causes | "Why did X happen?", "What caused Y?" |
| **Temporal** | TE | Lag variables, decay rates, precedent logic, chronological strictness | Temporal hallucination — confusing past/future, losing event sequence | "When will X complete?", "What happened before Y?" |
| **Spatial** | SP | Boundary enforcement, topology validation, dimensional constraints | Physical impossibilities — routes through walls, overlapping resources | "Where should X go?", "How do these components connect?" |
| **Simulation** | SI | Feedback loops, domino-effect tracking, systems archetypes | Single-step myopia — cannot model downstream consequences | "What happens if we change X?", "Model the impact of Y" |
| **Abstraction** | AB | Category enforcement, ontological boundaries, dimensionality control | Category errors — conflating metaphors with mechanisms, merging unrelated concepts | "What do these have in common?", "Classify X" |
| **Metacognition** | MC | Self-monitoring, contradiction detection, loop termination | Hallucination spirals — cannot detect own degradation, infinite regression | "Is my reasoning consistent?", "Am I making progress?" |

**51-54 abilities per domain, 311 total.** Each ability passed a 3-litmus test: must be a cognitive operation (not domain knowledge), must be LLM-executable (no external tools), must be domain-agnostic (works across subjects).

### What's Inside an Ability: The 4 Control Surfaces

Every ability contains 4 independent control surfaces that shape how the LLM reasons:

#### 1. Amplification (What to Activate)

An array of 2-4 compound reasoning signals. Positive attractors that pull the model toward specific patterns.

```
Amplify: depth_first_root_search; n_whys_traversal; systemic_fix_extraction
```

Effect: **Additive** — tells the model "also consider this." Broad, model can satisfy shallowly.

#### 2. Suppression (What to Block)

An array of 1-3 compound failure mode terms. Actively penalizes specific reasoning failures.

```
Suppress: symptom_treatment_bias; surface_level_stop
```

Effect: **Multiplicative** — tells the model "reject any output exhibiting this." Narrow, blocks entire failure class. **This is the most important control surface.** In testing, suppression-only payloads consistently outperformed amplification-only.

Why: "Do NOT treat symptoms as causes" forces deeper reasoning than "Find the root cause." Negative constraints are more specific than positive instructions.

#### 3. Cognitive Style (How to Reason)

A single semantic persona anchor that sets the methodology.

```
Style: root_cause_isolation
```

Examples: `bayesian_inference`, `meta_debugging`, `checkpoint_synchronization`, `meta_immunity`

#### 4. Reasoning Elasticity (How Far to Explore)

Controls the exploration-exploitation tradeoff:

| Value | Behavior | Use When |
|:------|:---------|:---------|
| `zero_drift` | Refuse to explore beyond immediate evidence | Auditing, compliance, debugging |
| `conservative` | Evidence-bound, cautious extrapolation only | Production incident analysis |
| `adaptive` | Balanced exploration within logical constraints | **Default for most tasks** |
| `high_variance` | Broad hypothesis generation, accepts uncertainty | Strategy, scenario planning |
| `max_entropy` | Unconstrained creative exploration | Brainstorming (pair with Metacognitive fallback) |

### The Reasoning Topology (DAG)

Every ability carries a **reasoning topology** — a directed acyclic graph (DAG) that encodes step-by-step execution with explicit decision points. 23 topology types across 5 families.

The DAG contains 3 node types:

| Node Type | Notation | What It Does | Example |
|:----------|:---------|:------------|:--------|
| **S-nodes** (Steps) | `[OP-1]`, `[OP-2]` | Sequential operations | "Identify the failure point and trace backward" |
| **G-gates** (Decisions) | `[GATE-G1]` | Binary conditions that route execution | "Output follows from input? → YES: next → NO: flag divergence" |
| **M-nodes** (Reflection) | `[REFLECT-M1]` | Meta-cognitive pause — LLM exits DAG, observes its own reasoning, can abandon and re-enter | "Am I making progress? → IF FAILING: reason freely, then re-enter at OP-1" |

**M-nodes are the breakthrough.** 84 of 311 abilities (27%) include them. They let the LLM pause structured execution, observe whether its approach is working, and if failing, exit the DAG for freeform reflection before re-entering. This prevents rigid procedures from forcing bad reasoning.

### How Routing Works

When you send a query, the API:

1. Embeds your query into a high-dimensional vector space
2. Runs **parallel hybrid search** — dense (semantic) + sparse (lexical) — across all 311 abilities
3. Returns the best-matching ability regardless of domain

Query specificity drives routing precision:

| Send This | Not This | Domain Activated |
|:----------|:---------|:-----------------|
| "Identify why customer churn increased 30% in Q3 after the pricing change" | "Help me analyze this" | **Causality** — root-cause tracing |
| "When will this migration complete based on current velocity?" | "How long will it take?" | **Temporal** — duration estimation, lag modeling |
| "Validate that these two services don't have conflicting resource claims" | "Check the services" | **Spatial** — boundary enforcement, conflict detection |
| "Model the downstream consequences of removing the rate limiter" | "Think about this problem" | **Simulation** — feedback loops, domino effects |
| "What do all our failing test cases have in common?" | "Look at the tests" | **Abstraction** — common structure extraction |
| "My agent hallucinates causal chains without tracing the mechanism" | "Fix my agent" | **Metacognition** — self-monitoring, contradiction detection |

**Key rule:** Send the agent's actual task description, not a meta-description of intent.

### Multi-Ability Composition (multi mode)

When using `multi` mode, 4 abilities are composed into a synergy chain:

| Role | What It Does | Synergy Edge |
|:-----|:------------|:-------------|
| **PRIMARY** | Best-matching ability, rendered in full | — |
| **DEPENDENCY** | Prerequisite ability — must execute first | `requires` edge |
| **AMPLIFIER** | Strengthens the primary's reasoning | `enhances` edge |
| **ALTERNATIVE** | Different analytical lens — challenges primary's conclusions | `fallback` edge |

Example flow: a Causal ability traces root cause → a Temporal ability projects recovery timeline → a Metacognitive ability monitors for contradictions between them.

Suppression vectors from all 4 abilities are deduplicated and merged. They compound — each one blocks a different failure class.

**Multi-mode response format:** The response is a single key (`multi_ability`) with one string value. All 4 abilities — PRIMARY, DEPENDENCY, AMPLIFIER, ALTERNATIVE — are composed into that single string, with merged suppression/amplification vectors at the end. You inject the whole string the same way as single mode. No extra parsing needed.

### What Single vs Multi Injection Looks Like

**Single mode** — 1 ability, pure signal injection (~2,000 chars):

```
[REASONING INJECTION]
Phase: MONITORING_LOOP | Confidence: ASSERT

[NEGATIVE GATE]
{failure pattern}

Step 1: ... Step 2: ... Step 3: ...

[REASONING TOPOLOGY]
S1:action → G1{test?} --yes→ S2 --no→ S3

[COGNITIVE PAYLOAD]
Amplify: ...
Suppress: ...
Style: ...

[FALSIFICATION TEST]
{falsification test}
```

**Multi mode** — 4 abilities, compound suppression (~4,000 chars):

```
[PRIMARY]
Phase: analysis | Confidence: CALIBRATE | Trigger: evidence_review_mismatch

[NEGATIVE GATE]
{failure pattern}

Step 1: ... Step 2: ... Step 3: ...

[REASONING TOPOLOGY]
...

[TARGET PATTERN]
...

[FALSIFICATION TEST]
...

[DEPENDENCY]
[NEGATIVE GATE] ...
Step 1: ... Step 2: ...
[FALSIFICATION TEST] ...

[AMPLIFIER]
...

[ALTERNATIVE]
...

[MERGED VECTORS]
Amplify: ...
Suppress: ...
```

---

## Code Examples

### Python — Minimal

```python
import requests

def get_reasoning(task: str, mode: str = "single") -> str:
    try:
        r = requests.post(
            "https://ejentum-main-ab125c3.zuplo.app/logicv1/",
            headers={
                "Authorization": "Bearer YOUR_API_KEY",
                "Content-Type": "application/json"
            },
            json={"query": task, "mode": mode},
            timeout=5
        )
        r.raise_for_status()
        data = r.json()
        key = f"{mode}_ability"
        payload = data[0].get(key, "") if isinstance(data, list) and data else ""
        if not payload:
            return ""
        return f"[REASONING CONTEXT]\n{payload}\n[END REASONING CONTEXT]"
    except (requests.RequestException, KeyError, IndexError):
        return ""  # Graceful degradation
```

### Python — LangChain LCEL

```python
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "{reasoning_context}\n\nYou are a senior analyst."),
    ("human", "{task}")
])

chain = (
    RunnablePassthrough.assign(
        reasoning_context=lambda x: get_reasoning(x["task"])
    )
    | prompt
    | ChatOpenAI(model="gpt-4")
)

result = chain.invoke({"task": "Why did our supply chain costs spike in Q3?"})
```

### Python — CrewAI

CrewAI uses `backstory` as its first-position context (equivalent to system message). Inject there:

```python
from crewai import Agent, Task, Crew

injection = get_reasoning("Analyze root cause of production failures")

# backstory is CrewAI's equivalent of the system message —
# it's the first context the LLM processes for this agent
analyst = Agent(
    role="Production Analyst",
    goal="Identify the root cause of system failures",
    backstory=f"{injection}\n\nYou are a production analyst.",
    llm=your_llm
)
```

### Python — Claude tool_use

```python
import anthropic

client = anthropic.Anthropic()

# Step 1: Get reasoning ability
injection = get_reasoning("Evaluate tradeoffs between microservice and monolith architecture")

# Step 2: Inject into system message
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=4096,
    system=f"{injection}\n\nYou are a senior software architect.",
    messages=[
        {"role": "user", "content": "Should we split the billing service into its own microservice?"}
    ]
)
```

### TypeScript

```typescript
async function getReasoning(task: string, mode: string = "single"): Promise<string> {
  try {
    const r = await fetch("https://ejentum-main-ab125c3.zuplo.app/logicv1/", {
      method: "POST",
      headers: {
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ query: task, mode })
    });
    if (!r.ok) return "";
    const data = await r.json();
    const key = `${mode}_ability`;
    const payload = Array.isArray(data) && data[0]?.[key] || "";
    return payload ? `[REASONING CONTEXT]\n${payload}\n[END REASONING CONTEXT]` : "";
  } catch {
    return "";  // Graceful degradation
  }
}
```

### Python — Multi-Turn Re-Injection

The guide says "re-inject per turn." Here's how:

```python
def agent_loop(tasks: list[str], llm_client):
    """Each turn gets a fresh injection matched to that turn's task."""
    conversation = []

    for task in tasks:
        # Fresh injection per turn — not reusing the first one
        injection = get_reasoning(task)

        response = llm_client.chat(
            system=f"{injection}\n\nYou are a senior analyst.",
            messages=conversation + [{"role": "user", "content": task}]
        )

        conversation.append({"role": "user", "content": task})
        conversation.append({"role": "assistant", "content": response})

    return conversation
```

Why per-turn: Turn 1 might need Causal reasoning ("why did this fail?"), Turn 2 might need Temporal ("when will it recover?"), Turn 3 might need Metacognitive ("is my analysis consistent?"). One static injection forces all turns into the same mode.

### cURL

```bash
curl -X POST "https://ejentum-main-ab125c3.zuplo.app/logicv1/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "Why did conversion drop 40% after checkout redesign?", "mode": "single"}'
```

---

## The Universal Integration Loop

Regardless of framework, every integration follows this pattern:

```
FOR each sub-task the agent needs to execute:
    1. SEND the task description to POST /logicv1/
    2. RECEIVE the reasoning ability
    3. QUARANTINE: validate response (non-empty, correct fields, relevant)
    4. FORMAT into [REASONING CONTEXT]...[END REASONING CONTEXT]
    5. INJECT into SYSTEM message, BEFORE task instructions
    6. EXECUTE the task with augmented reasoning
    7. VERIFY output against [FALSIFICATION TEST]
    8. If FAILED → re-query with failure description → retry (max 2)
```

---

## Key Insight: Suppression > Amplification

The `Suppress:` field is more important than the `Amplify:` field. Suppression signals block the model's natural tendency to:

- Stop at the first plausible answer
- List symptoms instead of tracing causes
- Hedge instead of committing to falsifiable conclusions
- Accept partial data as sufficient

RA²R doesn't make the agent smarter. It prevents the agent from being lazy.

---

## Choosing the Right Mode

The 2 modes serve different purposes. Choosing the right one based on what your agent needs maximizes the value of injection.

### Decision Framework

Ask one question: **Does my agent need one reasoning lens or multiple?**

#### `single` — When your agent needs to GET THE RIGHT ANSWER

Best for: debugging, root cause analysis, classification, diagnosis, decision-making, any task with a correct/incorrect outcome. Plan: Ki (starter).

Why it works: Delivers the highest density of suppression signals per token. Blocks cognitive shortcuts (premature conclusions, forward momentum bias, surface-level stopping) without constraining how the agent reasons. The agent stays flexible while avoiding its worst tendencies.

Example use cases:
- Agent investigating a production incident
- Agent classifying support tickets
- Agent evaluating whether a hypothesis is correct
- Agent making a go/no-go decision

#### `multi` — When your agent needs MULTIPLE PERSPECTIVES

Best for: complex tasks spanning multiple reasoning challenges, tasks where no single analytical lens is sufficient. Plan: Haki (premium).

Why it works: 4 abilities from different domains, each contributing its own suppression vector. The merged vectors block more failure modes simultaneously than any single ability. Compound suppression covers cross-dimensional reasoning.

Example use cases:
- Agent analyzing why a system failed AND estimating recovery timeline
- Agent evaluating a strategy with multiple stakeholder perspectives
- Agent assessing risk across technical, financial, and operational dimensions
- Agent producing an incident post-mortem report requiring multi-angle analysis

### Quick Reference

| I need my agent to... | Use | ~Size | Plan |
|:----------------------|:----|:------|:-----|
| Get the right answer | `single` | ~2,000 chars | Ki (starter) |
| Cover multiple angles | `multi` | ~4,000 chars | Haki (premium) |

### Rule of Thumb

If you're unsure, use `single`. It has the highest correctness improvement per token and the smallest injection size. Switch to `multi` when your task genuinely spans multiple reasoning dimensions.

---

## Benchmark Context: These Results Were Tested Against a Frontier Thinking Model

All benchmark results were produced against **Claude Opus 4.6** — Anthropic's most capable reasoning model with extended chain-of-thought, multi-step planning, and native self-correction.

The baseline accuracy of 69.7% reflects Opus 4.6's full native reasoning capability. The +7.1pp lift was achieved by augmenting a model that already reasons well. On multi-step abductive reasoning, Opus 4.6 natively solved only 20% of tasks correctly — RA²R injection raised this to 60%.

**RA²R is not a crutch for weak models.** It is a performance amplifier for any model, including frontier ones. The suppression signals block cognitive shortcuts that are architectural properties of transformer-based language models — forward momentum bias, premature conclusion, surface-level stopping — failure modes that persist even in the most capable thinking models.

If RA²R improves the strongest available model, the improvement on smaller or less capable models is expected to be equal or larger.

---

## Anti-Patterns (What NOT to Do)

| Anti-pattern | Why it fails |
|:-------------|:------------|
| Send a multi-step task as one query | Different sub-tasks need different reasoning domains — decompose first |
| Inject after task instructions | Payload loses attention priority |
| Inject into user message | Model treats it as data, not as operational constraint |
| Skip response validation | Corrupted or irrelevant payloads degrade output |
| Use one injection for all turns | Payloads degrade over long contexts — re-inject per turn |
| Ignore the Inhibit signals | Suppression produces the largest measurable improvement |
| Treat the API as a dependency | Always wrap with timeout + fallback. The API enhances, it's not required. |
