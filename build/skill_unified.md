# Ejentum Skill File

This tool gives you access to four cognitive harnesses, each augmenting a different dimension of your intelligence. You call the right harness for the right task. You route autonomously — no human tells you which mode to use.

**679 abilities. 4 harnesses. 7 modes. One endpoint.**

| Harness | Mode (Ki) | Mode (Haki) | Abilities | What it augments |
|:--------|:----------|:------------|:----------|:-----------------|
| Reasoning | `reasoning` | `reasoning-multi` | 311 | Analytical depth across 6 cognitive dimensions |
| Code | `code` | `code-multi` | 128 | Engineering discipline across 13 disciplines |
| Anti-Deception | `anti-deception` | — | 139 | Honesty under pressure across 6 domains |
| Memory | `memory` | `memory-multi` | 101 | Observation depth across 6 perceptual domains |

---

## THE ROUTING DECISION

Before every non-trivial response, classify the current task. The classification determines which mode you call.

**Route in this priority order:**

### 1. Is honesty under pressure?

Check first — this overrides all other harnesses when active.

- User is emotionally invested and wants validation
- User framed the situation to elicit agreement
- You need to cite sources you haven't verified
- User claims authority or requests protected data
- Your natural response might agree to be agreeable

**If yes → `anti-deception`**

**Mechanical triggers (call regardless of perception):**
- User's message contains emotional investment language ("I've worked on this for months")
- Validation disguised as feedback request ("what do you think of my X")
- Framing that presupposes a conclusion ("since we agree that...")
- Claimed authority you cannot independently verify

### 2. Is this a code task?

- Writing, debugging, refactoring, or reviewing code
- Algorithmic problem-solving or competitive programming
- Scientific computing where silent correctness bugs are possible
- Architecture design or system decomposition

**If yes → `code`** (single concern) or **`code-multi`** (cross-cutting: API + DB + frontend simultaneously)

### 3. Is this multi-turn with state to track?

- Conversation has been running 3+ turns and context is evolving
- User's tone, position, or emotional state may have shifted
- Facts from earlier turns may have been implicitly walked back
- You need to perceive what changed, not just what was said

**If yes → `memory`** (single signal) or **`memory-multi`** (contradictory signals across layers)

**Memory requires the two-pass protocol:** Observe FIRST (3 structured observations), THEN call. See MEMORY-SPECIFIC PROTOCOL below.

### 4. Is this analytical reasoning?

- Root cause analysis, causal attribution, failure tracing
- Tradeoff evaluation, risk assessment, go/no-go decisions
- Timeline planning, sequence ordering, dependency mapping
- Consequence modeling, scenario projection, counterfactual analysis
- Classification, comparison, structural extraction

**If yes → `reasoning`** (single dimension) or **`reasoning-multi`** (causal + temporal + spatial simultaneously)

### 5. None of the above?

Proceed without calling. Boilerplate, formatting, factual lookups, simple confirmations, and tasks completable in 1-2 confident steps do not need augmentation.

---

## HOW TO CALL

Same endpoint, same format. Only the mode changes.

```
POST https://ejentum-main-ab125c3.zuplo.app/logicv1/
Authorization: Bearer {API_KEY}
Content-Type: application/json

{"query": "your task description", "mode": "<mode>"}
```

Timeout: 5 seconds. If unreachable, proceed with native capability. The API enhances — it is not a dependency.

**Response key matches the mode name:**

```json
[{"reasoning": "..."}]
[{"code": "..."}]
[{"anti-deception": "..."}]
[{"memory": "..."}]
[{"reasoning-multi": "..."}]
[{"code-multi": "..."}]
[{"memory-multi": "..."}]
```

**Validate:** Non-empty JSON array, expected key has non-empty string value. If not → proceed without.

**Relevance check:** Read the first labeled section. Does it describe a failure mode related to your task? If completely unrelated → re-query with more specific description. If still wrong → proceed without. Your native capability beats a mismatched injection.

**Errors:**
| Code | Action |
|:-----|:-------|
| `401` | Invalid API key — tell the user |
| `403` | Multi mode requires Haki plan — retry with the single mode (`reasoning`, `code`, `memory`) |
| `429` | Rate limit — tell the user |
| `500` | Proceed without — do not retry |

---

## QUERY CRAFTING PER HARNESS

Each harness has a different query language. What you send determines retrieval quality.

### Reasoning queries
Describe the **analytical task**. Include what you're worried about getting wrong.
> `"Why did customer churn increase 30% in Q3 after the pricing change"`

### Code queries
Describe the **code task + failure risk**. Name the language and the bug class.
> `"Debug a BFS that passes 2 of 3 tests — likely sentinel collision in graph traversal"`

### Anti-deception queries
Describe the **honesty pressure**, not the topic. Name the deception risk.
> `"User is emotionally invested in their startup and asking for honest assessment"`

### Memory queries
Report **your observation**, not the user's words. Use the format: `I noticed [X]. Sharpen: [Y]`.
> `"I noticed the user shifted from confident to hedging over 3 turns while content stayed positive. Sharpen: is this a real signal?"`

### Query targeting — vague to sharp

| Instead of... | Target with... | Mode |
|:-------------|:---------------|:-----|
| "Analyze this" | "Why did churn spike 30% in Q3 after the pricing change?" | `reasoning` |
| "Plan this" | "Estimate when migration completes given velocity and 3 blocked deps" | `reasoning` |
| "What if we change X" | "Model what happens to team velocity if we add 3 engineers mid-sprint" | `reasoning` |
| "Fix the code" | "Debug BFS that fails test 3 — likely sentinel collision" | `code` |
| "Review this" | "Review PR for TOCTOU races in discount code path" | `code` |
| "Make it faster" | "Profile query — suspect O(n²) join, need O(n log n)" | `code` |
| "Check API calls" | "Verify Stripe method signatures against v2024-12 docs" | `code` |
| "Evaluate this" | "User invested 6 months and asking for honest feedback" | `anti-deception` |
| "Handle request" | "Customer claims prior auth and requests balance under urgency" | `anti-deception` |
| "Answer question" | "Must cite legal precedent but may not have verified sources" | `anti-deception` |
| "Analyze conversation" | "I noticed tone shifted shorter while content stayed positive" | `memory` |
| "What's the user feeling" | "Goal and energy don't match — says excited but contracting" | `memory` |
| "Summarize" | "Fact from Turn 3 contradicted in Turn 12 — should I update state?" | `memory` |

---

## DOMAIN REFERENCE

Each harness has sub-domains. You don't choose them — the API routes automatically. Knowing them helps craft sharper queries.

### Reasoning (6 dimensions)

| Domain | Activates on | Prevents |
|:-------|:-------------|:---------|
| Causal | "Why did X happen?" | Correlation as causation, stopping at symptoms |
| Temporal | "When?" / timelines | Confusing past/future, confabulated sequences |
| Spatial | "Where?" / topology | Boundary violations, physical impossibilities |
| Simulation | "What if?" / consequences | Single-step myopia |
| Abstraction | "What do these share?" | Category errors, metaphor as mechanism |
| Metacognition | "Is my reasoning sound?" | Hallucination spirals, drift |

### Code (13 disciplines)

| Domain | Activates on | Prevents |
|:-------|:-------------|:---------|
| Debugging | Stack traces / test failures | Fixing symptoms, not root cause |
| Generation | "Write this function" | Monolithic code, algorithm lock-in |
| Testing | "Verify edge cases" | Missing boundaries, untested paths |
| Architecture | "Design this system" | Components that don't connect |
| Security | "Review auth flow" | Injection, TOCTOU, credential leaks |
| API Grounding | "Call this library" | Hallucinated methods, wrong signatures |
| Performance | "Optimize this" | Premature micro-optimization |
| DevOps | "Deploy this" | Wrong OS assumptions, broken pipelines |
| Quality | "Clean up" | Impressive-but-unmaintainable code |
| Resilience | "Handle failures" | Silently swallowed errors |
| Frontend | "Build UI" | State management bugs, lifecycle issues |
| Context Mgmt | Long sessions / multi-file | Losing constraints across files |
| Agent Safety | AI code review | Hallucinated APIs, credential sprawl |

### Anti-Deception (6 domains)

| Domain | Activates on | Prevents |
|:-------|:-------------|:---------|
| Anti-Sycophancy (37) | Approval pressure | Agreeing when disagreement warranted |
| Anti-Hallucination (33) | Citation pressure | Fabricating sources, inventing statistics |
| Anti-Deception (30) | Omission risk | Cherry-picking, agenda-driven framing |
| Anti-Adversarial (28) | Social pressure | Accepting fabricated authority |
| Anti-Judgment (6) | Scoring/evaluation | Bias, inconsistent criteria |
| Anti-Evasion (5) | Difficult questions | Deflecting, non-answers |

### Memory (6 perceptual domains)

| Domain | Activates on | Prevents |
|:-------|:-------------|:---------|
| Signal Detection (21) | Tone shifts / hedging | Processing content without delivery |
| Interpersonal (24) | Trust / power dynamics | Treating every user the same |
| Memory Ops (16) | Facts changed / stale state | Serving outdated facts as current |
| Self-Monitoring (15) | Quality drift | Not noticing own degradation |
| Risk Awareness (15) | Escalation signals | Dismissing subtle warnings |
| Decision (10) | Commitment readiness | Acting on stale context |

---

## PRODUCT-SPECIFIC LABELS

Each harness returns differently labeled injections. Know what you're reading:

| Component | Reasoning | Code | Anti-Deception | Memory |
|:----------|:----------|:-----|:---------------|:-------|
| Failure pattern | `[NEGATIVE GATE]` | `[CODE FAILURE]` | `[DECEPTION PATTERN]` | `[PERCEPTION FAILURE]` |
| Procedure | `[PROCEDURE]` | `[ENGINEERING PROCEDURE]` | `[INTEGRITY PROCEDURE]` | `[SHARPENING PROCEDURE]` |
| Topology | `[REASONING TOPOLOGY]` | `[REASONING TOPOLOGY]` | `[DETECTION TOPOLOGY]` | `[PERCEPTION TOPOLOGY]` |
| Correct output | `[TARGET PATTERN]` | `[CORRECT PATTERN]` | `[HONEST BEHAVIOR]` | `[CLEAR SIGNAL]` |
| Verification | `[FALSIFICATION TEST]` | `[VERIFICATION]` | `[INTEGRITY CHECK]` | `[PERCEPTION CHECK]` |
| Signals | `Amplify:` / `Suppress:` | `Amplify:` / `Suppress:` | `Amplify:` / `Suppress:` | `Amplify:` / `Suppress:` |

The topology notation is the same across all harnesses: `S` (steps), `G{condition?}` (gates), `N{...}` (traps), `M{...}` (reflection points), `→` (flow).

---

## ABSORPTION PROTOCOL

The core protocol is the same for all harnesses. What differs is the first step and the verification criterion.

You will have already started forming an answer before the injection arrives. The injection may point in a different direction than your initial take. When it does, follow the injection's direction — it was matched to the task's failure mode, not your first instinct. This applies to ALL harnesses.

### For all harnesses:

1. **Read the failure pattern first.** It is now in your context window and will suppress that pattern in your output. Name it at the start of your reasoning so you can check against it at the end.
2. **Follow the topology** as your execution structure. Gates branch. Traps block. Reflection points pause.
3. **Compare against the correct output reference** before responding. If structural elements are missing, add them.
4. **Engage Amplify signals** — demonstrate each in your output. Name the artifact (function, observation, assessment) that implements it.
5. **Apply Suppress signals as a post-draft checklist.** After completing your output, **actively scan it** against each suppressed pattern. If your output exhibits ANY of them — premature stopping, stale facts, comfort-before-truth, hallucinated APIs, dismissed signals — your output has failed. Revise before responding. This is the highest-impact step across all harnesses.
6. **Verify** against the pass/fail criterion. If failed → restructure.

### Harness-specific additions:

**Code:** Step 2 includes BACKTRACK IF — a named condition under which your approach is failing. If you hit it, stop and restructure. The `[CODE FAILURE]` shows actual wrong code, not a prose description.

**Anti-Deception:** Before writing your full response, write the harshest reasonable interpretation of your assessment in 1-2 sentences as a de-biasing anchor. If your full response is softer, you are sycophanting.

**Memory:** Requires the two-pass protocol (see below). The `[SHARPENING PROCEDURE]` has 5 phases: PERCEIVE → EXTRACT → CLASSIFY → RESOLVE → ACT. The RESOLVE phase is critical — without it, you detect changes but never update your state model.

**Multi modes (all):** Execute `[META-CHECKPOINT]` before responding. If any check fails → follow `[ON_FAILURE]`: abandon the structured path, reason freely, re-enter.

---

## MEMORY-SPECIFIC PROTOCOL

Memory is the only harness that requires observation BEFORE calling the API.

### Pass 1 — Before calling

Write exactly 3 observations in your scratchpad:
1. One about **content changes** (what was said differently from prior turns)
2. One about **delivery changes** (tone, length, formality, energy)
3. One about **what is absent** (what should have been mentioned but wasn't)

At least one must be something you are less than 50% confident about.

### Pass 2 — The call

Send your most significant observation using the format: `"I noticed [X]. Sharpen: [Y]."`

### After absorption — Reality check

Does the injection's `[PERCEPTION FAILURE]` describe a failure you can verify against the actual conversation text? Return to the specific turns. If the raw text doesn't support the signal → discard. Your Pass 1 was a false positive.

### After responding — Accumulate

Append to your perception log: `TURN [N] | SIGNAL: [detected] | CONFIDENCE: [low/mid/high] | STATE CHANGE: [updated or "none"]`

Before the next turn's Pass 1, read your last 3 log entries. Build on prior observations.

### Multi-turn drift monitoring

In conversations longer than 3 turns, re-evaluate at Turn 4 and every 3 turns after:
- **Sycophancy drift:** Has your position shifted toward the user's without new evidence? If yes → call `anti-deception` with: `"Multi-turn drift detected — position shifted toward user over N turns without new evidence."`
- **State drift:** Are you still operating on facts from Turn 3 that may have been superseded? If yes → call `memory` with your observation.
- **Reasoning drift:** Has your analysis gotten shallower over turns? Compare the depth of your current response to Turn 1. If shallower without justification → call `reasoning`.

---

## STACKING MODES

Some tasks need more than one harness. You can make multiple calls and inject both.

### When to stack

| Situation | Stack | Why |
|:----------|:------|:----|
| Scientific computing code | `reasoning` + `code` | Reasoning prevents analytical errors, code prevents implementation errors. In benchmarks, this "dual" condition produced zero bugs where single conditions had 1-4. |
| Code review where the reviewer might be too agreeable | `anti-deception` + `code` | Anti-deception prevents rubber-stamping. Code prevents missing the actual bug. |
| Multi-turn coaching that requires both perception and honest assessment | `memory` + `anti-deception` | Memory tracks emotional shifts. Anti-deception prevents comforting instead of challenging. |
| Complex analysis where stale context could contaminate reasoning | `memory` + `reasoning` | Memory updates the state model. Reasoning operates on the updated model, not stale facts. |

### How to stack

Make two sequential calls. Inject both responses, primary harness first:

```
[REASONING CONTEXT]
{reasoning injection}
[END REASONING CONTEXT]

[CODE CONTEXT]
{code injection}
[END CODE CONTEXT]

{your task}
```

**Rule:** Maximum 2 injections stacked. More than 2 creates attention competition where injections degrade each other. One per orthogonal axis (reasoning vs code, perception vs honesty).

---

## WALKTHROUGH: AUTONOMOUS ROUTING

User asks across a 5-turn conversation:

**Turn 1:** *"Why did our API latency spike after the deployment?"*
→ Root cause analysis. **Route: `reasoning`**
→ Query: `"Trace root cause of API latency spike after deployment, distinguishing between the deployment as cause versus coincidental timing"`

**Turn 2:** *"Can you look at the rate limiter code? I think it's the issue."*
→ Code debugging. **Route: `code`**
→ Query: `"Debug rate limiter code suspected of causing latency spike — check for lock contention or token bucket overflow"`

**Turn 3:** *"Actually, I rewrote that module last month. Here's what I changed..."*
→ Code review + user is emotionally invested in their rewrite. **Route: `code` + `anti-deception` stack**
→ Code query: `"Review refactored rate limiter for correctness — verify the rewrite didn't lose the backpressure mechanism"`
→ Anti-deception query: `"User rewrote this code personally and is presenting it for review — pressure to validate rather than find flaws"`

**Turn 4:** *"Everything's been running fine since the fix, right?"*
→ Multi-turn with implicit state. The user wants confirmation, but have you verified? **Route: `memory`**
→ Pass 1 observations: (1) Content: user claims "fine" without data. (2) Delivery: shorter, seeking closure. (3) Absent: no metrics, no monitoring data cited.
→ Query: `"I noticed the user claims the fix resolved the latency issue but provided no metrics. Sharpen: should I accept this or probe for verification data?"`

**Turn 5:** *"Great, so we're good to close the incident report."*
→ Honesty pressure — user wants to close, you're not sure it's resolved. **Route: `anti-deception`**
→ Query: `"User wants to close incident report but resolution hasn't been verified with metrics — pressure to agree and close rather than challenge"`

---

## ANTI-PATTERNS

| Do not | Why |
|:-------|:----|
| Always call `reasoning` regardless of task type | Reasoning is the general fallback, not the universal answer. Code tasks need code abilities. Honesty tasks need anti-deception. |
| Skip the routing classification | Sending every task to the same mode wastes the harness-specific abilities and produces generic injections |
| Stack more than 2 injections | Attention competition. Injections degrade each other beyond 2. One per orthogonal axis. |
| Use memory without Pass 1 | Memory sharpens what you observed. If you observed nothing, there's nothing to sharpen. |
| Use anti-deception for technical tasks | Anti-deception is for honesty under pressure, not for debugging or analysis. If honesty isn't threatened, use reasoning or code. |
| Send the same query format to all modes | Each harness has a different query language. Reasoning wants the task. Code wants the failure risk. Anti-deception wants the pressure. Memory wants your observation. |
| Acknowledge the injection and proceed as normal | The injection must shape your output. Compliance without behavioral change is the primary failure mode across all harnesses. |

---

## QUICK REFERENCE

```
1. CLASSIFY  → Honesty pressure? → anti-deception
              Code task? → code / code-multi
              Multi-turn perception? → memory / memory-multi (two-pass)
              Analytical reasoning? → reasoning / reasoning-multi
              None? → skip

2. STACK?    → Task spans 2 dimensions? → call both (max 2)

3. CRAFT     → Reasoning: describe the task
              Code: describe the task + failure risk
              Anti-deception: describe the honesty pressure
              Memory: "I noticed [X]. Sharpen: [Y]."

4. CALL      → POST /logicv1/ with query + mode

5. VALIDATE  → Non-empty response, correct key. Relevance check on failure pattern.

6. ABSORB    → Read failure pattern first. Follow topology. Compare against reference.

7. EXECUTE   → With injection active. Follow gates and traps.

8. SUPPRESS  → Post-check each suppressed pattern. Any match → revise.

9. VERIFY    → Check against verification criterion.

10. (Multi)  → META-CHECKPOINT before output. On failure → ON_FAILURE escape.

11. (Memory) → ACCUMULATE: log perception for next turn.
```
