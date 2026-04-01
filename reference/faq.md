# FAQ

Straight answers from the builder who designed this system. If your question is not here, it should be. Reach out and we will add it.

## Getting Started

### Do I need to write code?

No. If you use n8n or Make.com, you can integrate with HTTP request nodes and zero code. If you use LangChain, CrewAI, or other frameworks, see the [Integrations guide](../build/integrations.md) for copy-pasteable code examples.

### How much does it cost?

Free tier: 100 calls, no credit card required. Ki: €19/month for 10,000 calls (single-ability mode). Haki: €49/month for 50,000 calls (multi-ability mode with compound scaffolds). [See pricing](https://ejentum.com/pricing).

### How long does setup take?

Under 10 minutes. Create an account, generate an API key from your dashboard, and make your first API call. The [Quickstart](../getting-started/quickstart.md) walks through it step by step.

## Understanding Ejentum

### How is this different from just writing a better system prompt?

System prompts are static, monolithic, and degrade at scale. Ejentum replaces a 5,000-token system prompt with a compact, structured payload injected dynamically at runtime.

The key difference: the payload contains both amplification signals (what to think about) and [suppression signals](../getting-started/concepts.md#why-suppression-matters-more-than-amplification) (what failure modes to block). A system prompt says "be careful." A suppression signal says "reject any output that exhibits `symptom_treatment_bias`." In our testing, this produces measurably sharper reasoning.

### Is this just RAG?

No. RAG retrieves *information*: documents, facts, data chunks. The agent still decides how to reason about that information using whatever patterns it learned during training.

RA²R (Reasoning Ability-Augmented Retrieval) retrieves *reasoning abilities*: a structured payload that governs how the agent thinks. See [Concepts](../getting-started/concepts.md) for the full explanation.

### Does this replace my LLM?

No. We are not a model. We are not a model provider.

You bring the engine (OpenAI, Anthropic, Mistral, Llama, or any instruction-following LLM). Ejentum provides the reasoning abilities that govern how that engine thinks — what patterns to follow, what failure modes to avoid. The system is model-agnostic. The cognitive operations (suppression, amplification, falsification checkpoints) work with any instruction-following model.

### How does this interact with chain-of-thought prompting?

They are complementary, not competing. Chain-of-thought tells the model to *show* its reasoning steps. Ejentum tells the model *which* reasoning steps to take and which failure modes to avoid during those steps.

Without Ejentum: "Let's think step by step" and the model picks whatever steps feel statistically probable.

With Ejentum: "Let's think step by step" + suppression of `surface_level_stop` + amplification of `depth_first_root_search`. The model's steps go deeper and avoid premature conclusions.

Use both. Ejentum structures the reasoning. Chain-of-thought makes it visible.

### Does this work with fine-tuned models?

Yes. The injection operates at the prompt level, not the model level. Any model that accepts a system prompt — including fine-tuned GPT-4, Llama variants, Mistral, and domain-specific models — is expected to respond to the structured reasoning payloads. Effectiveness scales with instruction-following capability; models with stronger instruction compliance produce sharper suppression response.

Fine-tuned models may respond *more* effectively because their base weights already encode domain knowledge. Ejentum adds the reasoning abilities on top.

## Compatibility

### Does this work with my framework?

If your framework can make an HTTP POST request and inject text into a prompt, it works with Ejentum. We have specific guides for [n8n](../build/integrations.md#n8n), [LangChain](../build/integrations.md#langchain--langgraph), [CrewAI](../build/integrations.md#crewai), [Claude Code](../build/integrations.md#claude-code--agent-sdk), and [agentic IDEs](../build/integrations.md#agentic-ides-cursor-windsurf-antigravity-codex). No SDK required.

### What about latency?

The scaffold retrieval happens before your agent generates a single token. The pipeline runs in under one second. No LLM is called during retrieval. Zero inference cost. The scaffold adds approximately 500 tokens (single mode) or 900 tokens (multi mode) to your agent's context.

### What if my agent doesn't need all six reasoning dimensions?

Start with the **Ki plan** (€19/month). It gives access to all 6 dimensions and all 311 abilities with single-ability injection (1 ability per call). If your tasks require cross-dimensional reasoning (e.g., causal analysis + temporal forecasting + metacognitive monitoring simultaneously), upgrade to **Haki** (€49/month) for multi-ability injection (4 synergized abilities per call with compound suppression).

### Can I use multiple abilities at once on the Ki plan?

No. Ki returns one ability per query: the highest-scoring cognitive scaffold for your task. If your task spans multiple reasoning dimensions, the strongest match wins.

For tasks that genuinely require cross-dimensional reasoning, the Haki plan returns 4 synergized abilities: a primary scaffold plus dependency, amplifier, and alternative frameworks with merged suppression vectors.

**Workaround on Ki:** Make multiple API calls with dimension-specific sub-queries and stack the injections manually. This works but loses the synergy composition, so the abilities may not be optimally combined.

## Using It Well

### How do I know which ability my query will trigger?

You don't need to. The hybrid search engine retrieves the optimal ability automatically across all six dimensions. You send a natural language task, and the API returns the structured payload your agent needs.

The response includes metadata that tells you which ability was selected. Log it to help debug routing over time.

### What if the router picks the wrong ability?

Three approaches:

1. **Improve your query.** The retrieval engine uses hybrid semantic + lexical search. Vague queries produce vague results. Send the full task description, not a summary. Include domain-specific keywords that signal the dimension you need.

2. **Inspect the response.** The response metadata shows which ability was selected. If a causal ability fires when you expected a temporal one, your query emphasizes causal language over temporal language.

3. **Rewrite and compare.** Send two versions of your query and compare the returned abilities. The delta tells you which keywords are driving the routing.

The router does not have a manual override. This is by design. If you already know which ability you need, you don't need automatic routing. The value is in the automatic matching for queries where the right reasoning mode is not obvious.

See the [query optimization guide](../build/api_reference.md#writing-better-queries) for concrete examples.

### How accurate is the retrieval?

Retrieval precision scales with query specificity. Explicit queries ("why did X cause Y after Z?") match with high confidence across Causal and Temporal dimensions simultaneously. Vague one-liners may return a general-purpose ability.

Best practice: send the full task description, not a condensed summary. The hybrid search engine matches your query both semantically and lexically.

### What happens when my query is ambiguous or spans multiple dimensions?

The hybrid search evaluates your query across all six dimensions simultaneously. For ambiguous queries, the highest-scoring dimension wins on Ki. On Haki, four synergized abilities are composed into a single scaffold. Sending your agent's full task description rather than raw user input produces significantly sharper retrieval.

### How do I handle multi-turn conversations?

Re-inject per turn. Each turn in a multi-turn conversation may require different reasoning abilities. Turn 1 might need root-cause analysis. Turn 2 might need temporal forecasting. Turn 3 might need metacognitive self-monitoring.

Call the Ejentum API with the current turn's task description, not the conversation history. Per-turn injection keeps the reasoning context fresh.

For long conversations (10+ turns), re-injection keeps the scaffold fresh. However, scaffolds resist decay more than plain instructions: on [ARC-AGI-3](https://ejentum.com/blog/arc-agi-3-benchmark-report) (25 sequential game actions), scaffold language persisted with a half-life of 24 steps and reasoning quality improved over time instead of degrading. Re-injection is still recommended per turn for optimal results, but a single scaffold injection does not vanish after 5 turns.

### What is the "lost in the middle" problem and how does Ejentum address it?

LLMs attend most strongly to content at the beginning and end of their context window (Liu et al., 2023). Ejentum addresses this with a compressed structured payload (less middle to get lost in), structured `[REASONING CONTEXT]` delimiters (distinct attention anchor), and [injection at the START](../build/integrations.md#the-injection-principle) of the system message. The [Cognitive Scaffolding Thesis](https://ejentum.com/blog/cognitive-scaffolding-thesis) models why: the scaffold's DAG notation occupies a structurally unique register in token space, receiving disproportionate attention weight that resists decay as task-specific tokens accumulate.

## Reliability and Trust

### Can I see all 311 abilities?

Yes. The [Abilities catalog](https://ejentum.com/abilities) contains the full reference: every ability, its reasoning scaffold, its dimension scores, and its synergy connections.

### What happens if the API is down?

Your agent continues functioning on native LLM reasoning — it just loses the reasoning injection. Ejentum is an enhancement layer, not a critical-path dependency. We architect for [graceful degradation](../build/integrations.md#graceful-degradation). The API runs on a global edge network, targeting 99.9% availability.

### Is my query data stored?

All traffic is logged for audit and debugging purposes at the gateway level. Query content is not used for training or shared with third parties.

### How do I measure if Ejentum is actually improving my agent?

Follow our [Evaluate guide](../getting-started/evaluate.md). In short: run 20+ tasks with and without injection, score on 4 signals (correctness, self-monitoring, verification, depth). In our benchmarks across 250 tasks (180 custom professional + 70 published academic), self-monitoring improved +132%, verification +85%, and correctness held steady or improved across 10 professional domains. On interactive multi-step tasks ([ARC-AGI-3](https://ejentum.com/blog/arc-agi-3-benchmark-report)), scaffold persistence and memory decay reversal were measurable across 25-step chains. Your results will vary, but the methodology is reproducible in one afternoon.

### What can't Ejentum do?

Honest boundaries:

- **It does not add domain knowledge.** If your agent fails because it lacks information (no access to your database, missing context), use RAG. Ejentum improves how the agent reasons about information it already has.
- **It operates at the prompt level.** It does not modify model weights, activations, or fine-tuning. It structures the prompt in ways that measurably improve reasoning output.
- **Retrieval precision depends on query quality.** Very short or ambiguous queries may not retrieve the optimal ability. Send the full task description for best results.
- **Suppression reduces failure rates; it does not guarantee zero failures.** LLMs are probabilistic. Ejentum steers the model away from specific failure modes, but cannot enforce absolute constraints.
