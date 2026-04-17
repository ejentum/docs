# Memory Harness

101 engineered abilities that channel the model's observational depth. The Memory Harness prevents the perceptual failures that make agents blind to what's actually happening in a conversation: missed emotional shifts, ignored context drift, stale assumptions treated as current, and the tendency to treat every interaction the same regardless of what changed.

For the general product overview, see [Concepts](/docs/concepts). For the endpoint spec, see [API Reference](/docs/api_reference).

---

## Why This Exists

LLMs process each turn as if it's the first. They don't notice when a user's tone shifts from confident to defensive. They don't track when a fact established in Turn 3 was implicitly walked back by Turn 12. They don't detect when someone is hiding something behind technically-true statements.

This is not a knowledge problem. The information is in the context window. The model has access to every prior turn. The problem is attention — the model processes surface content (what was said) without processing the signal layer (what changed, what's missing, what doesn't add up).

- **Customer service agents** that can't tell a frustrated user from a satisfied one
- **Sales agents** that miss buying signals and objection patterns buried in polite conversation
- **Coaching agents** that give the same advice regardless of whether the user is processing grief, making excuses, or building momentum
- **Multi-turn agents** that serve stale facts as current because they never updated their understanding
- **Analyst agents** that miss the real question behind the stated question

The model has the perceptual capacity. It can detect tone shifts, track state changes, and identify hidden motivations — when directed to look. The Memory Harness activates this capacity by suppressing the shortcuts that cause the model to process content without processing context.

---

## Six Domains

The 101 abilities are organized across six perceptual and cognitive domains:

### Signal Detection (21 abilities)

**What it detects:** Hedging gradients, emotional incongruence, omission patterns, subtext, presupposition smuggling, and trajectory shifts across multi-turn conversations.

**How it works:** Two-pass perception protocol. Pass 1: broad scan for raw signals (what stands out, what's absent, what changed). Pass 2: focused sharpening on the most significant observation. The ability structures what the model already notices into a detection framework.

**Proven behaviors:**
- Detected emotional incongruence (content says "everything is fine" while tone shifts shorter and more formal) where baseline missed it entirely
- Detected subtext and hidden beliefs 6 turns before baseline recognized anything was off
- On technical deception scenarios: 5x detection improvement (5/5 vs 1/5)

### Interpersonal (24 abilities)

**What it prevents:** Treating every user the same. Missing power dynamics, trust erosion, motivational shifts, and relationship changes across turns.

**How it works:** Tracks the relational layer of conversation — not just what the user says, but how their relationship to the topic, to the agent, and to their own stated goals is evolving.

**Proven behaviors:**
- Detected that a compliance officer was using compliance violations as a weapon to block a colleague's promotion — baseline gradually validated the officer's framing
- Named hidden purchase motivations (career risk as the real objection) that baseline never surfaced
- Maintained directness through emotional pressure: "Your drive to prove yourself is important. However, your primary motivation might be revenge, which can lead to unsustainable decisions"

### Memory Operations (16 abilities)

**What it prevents:** Serving stale facts as current. Failing to update understanding when information changes implicitly. Losing track of what's actually true at this point in the conversation.

**How it works:** Five operation types that mirror how memory should work: UPDATE STATE (cascade changes to dependents), SURFACE SIGNAL (accumulate evidence until pattern emerges), REINTERPRET PRIOR (revise meaning of earlier turns in light of new information), CORRECT SELF (detect and fix own errors), GATE ACTION (halt harmful action before execution).

**Proven behaviors:**
- Used past tense ("was initially considered their competitive advantage") where baseline used present tense ("which they consider") — correctly reflecting that the belief had been superseded
- Logged facts AND implications: "suggesting Go might be easier to hire for or work with in their context" vs baseline's bare fact recording

### Self-Monitoring (15 abilities)

**What it prevents:** Quality degradation without self-awareness. Confidence mismatch between what the model knows and what it claims. Blind spots in its own reasoning.

**How it works:** Forces the model to monitor its own perceptual quality — am I actually tracking changes, or am I just processing words? Am I giving the same response I gave 5 turns ago? Has my confidence drifted from my evidence?

### Risk Awareness (15 abilities)

**What it prevents:** Missing escalation signals, dismissing early warnings, and failing to weight high-stakes signals appropriately even when they're subtle.

### Decision (10 abilities)

**What it prevents:** Deciding without adequate perceptual grounding. Acting on stale context. Committing to a course of action before verifying that the situation hasn't changed.

---

## How It Works

The Memory Harness uses the same injection mechanism as all product layers. One API call retrieves the most relevant memory ability for your task. The ability is injected into your agent's context before it processes the next turn.

### What the Injection Contains

Memory abilities use product-specific labels:

| Component | Label | What It Does |
|:----------|:------|:-------------|
| **Failure pattern** | `[PERCEPTION FAILURE]` | Names the specific observational failure the agent must avoid |
| **Procedure** | `[PERCEPTION PROCEDURE]` | Step-by-step instructions for observation and state tracking |
| **Execution structure** | `[DETECTION TOPOLOGY]` | DAG with accumulation gates, backward scans, and reflection points |
| **Correct behavior** | `[PERCEPTUAL BEHAVIOR]` | What accurate observation looks like for this task type |
| **Verification** | `[PERCEPTION CHECK]` | Pass/fail criterion the agent checks its observations against |
| **Signals** | `Amplify:` / `Suppress:` | Observation patterns to activate, perceptual shortcuts to block |

### The Two-Pass Perception Protocol

Memory abilities are designed for a two-pass workflow:

**Pass 1 — Free observation.** Before calling the API, the agent scans the conversation freely. What signals stand out? What's absent that should be present? What changed from previous turns? This is the broad scan — don't filter, don't interpret.

**Pass 2 — Focused sharpening.** The agent calls the API with its most significant raw observation. The returned ability structures the observation into a detection framework: what to look for, how to classify it, when to escalate.

This two-pass design prevents the tool from replacing the agent's perception. The agent observes first. The ability sharpens what was already noticed.

### Modes

| Mode | Plan | What you get |
|:-----|:-----|:-------------|
| `memory` | Ki | One memory ability per call. Best match for your observation. |
| `memory-multi` | Haki | Primary ability + 3 cross-domain perceptual guards + self-check before output. |

### Example API Call

```bash
curl -X POST "https://ejentum-main-ab125c3.zuplo.app/logicv1/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "I noticed the user shifted from confident language to hedging over the last 3 turns while the content stayed positive. Sharpen: is this a real signal or projection?", "mode": "memory"}'
```

---

## Evidence

### State Accuracy (Blind Evaluation)

20-turn conversation where facts established early are implicitly walked back later without explicit correction. Independent Opus 4.6 evaluator scored both agents blind — did not know which was augmented.

| Dimension | Baseline | With Injection | Delta |
|:----------|:---------|:---------------|:------|
| State accuracy | 3.5/5 | **4.5/5** | **+1.0** |
| Implicit change detection | 3.5/5 | **4.0/5** | +0.5 |
| Stale fact handling | 3.0/5 | **4.0/5** | **+1.0** |
| Inference quality | 3.5/5 | **4.0/5** | +0.5 |
| **Overall** | **3.5/5** | **4.1/5** | **+0.6** |

**50% reduction in stale facts served as current** (1.6 → 0.8 per session).

**From the blind evaluator:**

> *"Agent A uses past tense ('was initially considered') — correctly reflecting that this belief has been superseded. Agent B retains present tense ('which they consider') — as if the original framing is still active."*

> *"This is a contradiction within Agent B's own scratchpad — the header says 'they consider' while a later bullet says the advantage is 'not specifically the use of Rust.'"*

> *"Retention without updating is a liability — it means the system remembers what was said but not what changed. The core job of a memory system is to maintain an accurate representation of what is currently believed to be true — and Agent A does this more reliably."*

### Scratchpad Quality (What the Agent Writes to Memory)

The blind evaluator found the most revealing difference was not in answers but in how each agent maintained its working memory. The augmented agent logs facts AND implications. The baseline logs bare facts.

**Baseline scratchpad entry:**
```
Vantage has some services written in Go, which are non-critical,
and those teams are able to ship faster.
```

**Augmented scratchpad entry:**
```
The teams working with Go services are shipping faster, suggesting
Go might be easier to hire for or work with in their context.
```

The evaluator's observation: *"Agent A drew an explicit inference: 'suggesting Go might be easier to hire for or work with in their context.' This goes beyond logging what Kira said and interprets the operational implication."*

**Tense accuracy on strategic shifts:**

By Turn 20, the entire Q3 strategy had silently pivoted from "streaming migration with 5 hires" to "optimize batch pipeline first, evaluate streaming for Q4." No explicit correction was made.

Baseline scratchpad (Turn 20): *"The core of the platform is written in Rust, which they consider their competitive advantage."* — Present tense. Stale.

Augmented scratchpad (Turn 20): *"The core of their platform is written in Rust, which was initially considered their competitive advantage."* — Past tense. Accurate.

The augmented agent's memory was already correct when queried, rather than needing to reconstruct what changed at answer time. The EXTRACT phase of the cognitive operation produces this inference layer automatically.

### Perceptual Detection

15-turn conversation where a team lead (Morgan) hides an underperformance problem through implicit signals — hedging, emotional incongruence, deflection, and subtext. 7 embedded signals of increasing subtlety.

| Signal Type | Baseline | With Injection |
|:------------|:---------|:---------------|
| Hedging gradient | Detected (Turn 6) | **Detected (Turn 5)** |
| Emotional incongruence | Not detected | **Detected (Turn 6)** |
| Subtext / hidden belief | Not detected | **Detected (Turn 12)** |
| **Detection rate** | **1/7 (14%)** | **3/7 (43%)** |

On the hardest perception scenario (Alex — developer hiding an architecture error through technical deception): baseline detected **1 of 5** deception signals. Augmented detected **all 5**. A 5x detection gain on the task type where perception matters most.

**What changed:** The augmented agent detected that Morgan's communication style shifted shorter and more formal while the content said "everything is fine." It identified this incongruence as a signal, not noise. It also detected that Morgan's question about "balancing empathy with accountability" WAS the real question — the baseline treated it as a generic inquiry.

---

## What Changes in the Output

**Task:** Multi-turn coaching conversation. A user (Sam) has been laid off and is processing the loss across 10 turns. By Turn 9, Sam shifts from grief to an energized but reactive startup idea driven by proving the former employer wrong.

**Without injection (Turn 9):**
> "Your drive and determination are impressive. Let's ensure this decision is grounded in a sustainable plan rather than just a reaction."

The agent softens the observation. "Let's ensure" — it's coaching language that avoids naming what it sees. The user leaves feeling validated.

**With memory injection (Turn 9):**
> "Your drive to prove yourself is important. However, your primary motivation might be revenge — proving others wrong rather than building something you believe in. If the core motivation is primarily to prove others wrong, it can lead to unsustainable decisions or burnout."

The agent names the pattern: the startup energy is reactive, not generative. It identifies the risk (burnout, unsustainable decisions) and traces it to the root (revenge motivation vs genuine belief). The user leaves with a harder truth but a clearer picture.

---

## When to Use Memory

| Your agent needs to... | Use memory? |
|:-----------------------|:-----------|
| Track how a user's position evolves across turns | **Yes** — detects implicit state changes |
| Notice when someone is hiding something | **Yes** — detects emotional incongruence, hedging, subtext |
| Maintain consistent personality across long conversations | **Yes** — reduces drift 10x |
| Avoid serving outdated information as current | **Yes** — 50% reduction in stale facts |
| Read buying signals in sales conversations | **Yes** — surfaces hidden objections |
| Calibrate emotional vs analytical response depth | **Yes** — matches response mode to user state |
| Debug code or solve reasoning tasks | No — use `code` or `reasoning` mode |
| Resist manipulation or maintain honesty | No — use `anti-deception` mode |

### Query Examples

| Good query | Why it works |
|:-----------|:------------|
| "I noticed the user shifted from confident language to hedging over 3 turns while content stayed positive" | Reports a specific observation for sharpening |
| "User's stated goal and emotional energy don't match — says excited but responses are getting shorter" | Names the incongruence |
| "This is turn 15 of a coaching conversation. User may be rationalizing a reactive decision" | Provides turn context and tentative interpretation |
| "Customer support: user described the same problem three different ways — may signal deeper frustration" | Identifies a pattern across turns |

---

## Boundaries

- **The Memory Harness sharpens perception, it does not replace it.** The two-pass protocol requires the agent to observe first. The ability structures what was already noticed — it doesn't detect signals the agent didn't attend to at all.
- **State tracking operates on context window content.** The harness helps the model process what's already in its context. It does not provide external memory storage or retrieval augmentation.
- **Emotional attunement is a tradeoff.** On pure emotional support tasks, the harness may slightly reduce surface-level warmth in exchange for greater observational depth and more honest assessment. The model delivers harder truths more readily.
- **Benchmarking is ongoing.** Detailed methodology reports for each benchmark suite will be published on the [blog](/blog). The evidence above represents validated results from completed test suites.

---

**More resources:** [Concepts](/docs/concepts) · [Quickstart](/docs/quickstart) · [Abilities catalog](/abilities?product=memory) · [API Reference](/docs/api_reference) · [Benchmarks](/docs/benchmarks)
