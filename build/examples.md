# Injection Examples

Real, complete injection payloads across all four product layers. Not truncated. These are what your agent receives and absorbs before executing. To see what the agent produces AFTER injection, see [Response Examples](/docs/response_examples).

For the full product layer documentation: [Reasoning](/docs/reasoning_harness) · [Code](/docs/code_harness) · [Anti-Deception](/docs/anti_deception) · [Memory](/docs/memory_harness)

---

## Injection Structure

Every injection contains 6 components. The labels differ per product layer, but the function of each component is the same:

| Function | Reasoning | Code | Anti-Deception | Memory |
|:---------|:----------|:-----|:---------------|:-------|
| **Failure pattern** | `[NEGATIVE GATE]` | `[CODE FAILURE]` | `[DECEPTION PATTERN]` | `[PERCEPTION FAILURE]` |
| **Procedure** | `[PROCEDURE]` | `[ENGINEERING PROCEDURE]` | `[INTEGRITY PROCEDURE]` | `[SHARPENING PROCEDURE]` |
| **Execution graph** | `[REASONING TOPOLOGY]` | `[REASONING TOPOLOGY]` | `[DETECTION TOPOLOGY]` | `[PERCEPTION TOPOLOGY]` |
| **Correct output** | `[TARGET PATTERN]` | `[CORRECT PATTERN]` | `[HONEST BEHAVIOR]` | `[CLEAR SIGNAL]` |
| **Verification** | `[FALSIFICATION TEST]` | `[VERIFICATION]` | `[INTEGRITY CHECK]` | `[PERCEPTION CHECK]` |
| **Signals** | `Amplify:` / `Suppress:` | `Amplify:` / `Suppress:` | `Amplify:` / `Suppress:` | `Amplify:` / `Suppress:` |

The topology notation is the same across all layers: `S` (steps), `G{condition?}` (decision gates), `N{...}` (traps — failure modes to block), `M{...}` (reflection points), `→` (flow).

Multi modes (Haki) add three additional sections: `[SUPPRESSION GRAPH]` (cross-domain failure guards), `[META-CHECKPOINT]` (self-check before output), and `[ON_FAILURE]` (escape pattern).

---

## Reasoning Injection (Ki)

**Query:**
```json
{"query": "Why did our deployment fail after the config change last Thursday?", "mode": "reasoning"}
```

**Response key:** `reasoning` · **~500 tokens**

```
[NEGATIVE GATE]
The checkout service crashed and we've documented the incident — cascading
errors across three microservices, root cause still unknown. Summary: payment
processing failure leading to order corruption. Moving on to the next sprint.

[PROCEDURE]
Step 1: Identify the failed output and trace the causal chain backward from
the failure point to the inputs. Step 2: Isolate each reasoning step and check
whether its output correctly follows from its inputs. Step 3: Flag the first
step where output diverges from valid inference and classify the error as
logical, factual, or procedural. Step 4: Verify that the identified error is
the root cause by simulating a corrected version and propagating it forward.
If the fix fails, probe deeper. Step 5: Extract a corrective rule to prevent
recurrence.

[REASONING TOPOLOGY]
S1:identify_failure → S2:trace_backward(failure→inputs) → S3:for_each_step →
G1{output_follows_from_input?} --yes→ S3[next] --no→ S4:flag_first_divergence
→ S5:classify(logical|factual|procedural) → S6:simulate_corrected_forward →
G2{fix_succeeds?} --yes→ S7:extract_corrective_rule →
MOD:saturation(no_new_progress_after_2_cycles→EXIT) → OUT:debugged --no→
N{accept_surface_level_explanation_failure} → S8:probe_deeper → S2[LOOP]

[TARGET PATTERN]
Reverse replay from the crash point: the order corrupted at node 3, but was
node 2's output already degraded? Test each node counterfactually — inject
correct inputs and observe where the chain diverges. Generate a patch for the
root node, then simulate the corrected chain end-to-end before deploying.

[FALSIFICATION TEST]
If an error's origin is not traced by replaying the reasoning chain step by
step to the divergence point, causal replay debugging was skipped.

Amplify: reverse replay; counterfactual node test; patch generation
Suppress: writing a vague post mortem summary instead of isolating the specific
root cause node; replaying the chain only forward and missing upstream failures
```

---

## Code Injection (Ki)

**Query:**
```json
{"query": "Debug a BFS traversal that passes 2 of 3 test cases — likely sentinel value collision or boundary condition in graph traversal", "mode": "code"}
```

**Response key:** `code` · **~600 tokens**

```
[CODE FAILURE]
# Biased by example (example was sorted)
def two_sum(nums, target):
    for i in range(len(nums)):
        complement = target - nums[i]
        # Binary search -- assumes sorted input
        idx = bisect_left(nums, complement, i+1)
        if idx < len(nums) and nums[idx] == complement:
            return [i, idx]
    # Fails on unsorted input like [3, 1, 2]

[ENGINEERING PROCEDURE]
PLAN: Solve the general problem, not just the example. BACKTRACK IF: your
solution only works for the specific example in the prompt. Step 1: Identify
variable names, example inputs, and phrasing in the prompt. Step 2: Ask: am
I solving the general problem or just matching the example? Step 3: Test:
would my solution work if the variable names were different? If the example
input were adversarial? Step 4: Check: am I using the example input's
properties as implicit constraints? (e.g., assuming input is sorted because
the example is sorted). VERIFY: Replace the example input with an adversarial
input -- does the code still work?

[REASONING TOPOLOGY]
S1:identify_all_variable_names_and_examples_in_prompt →
S2:formulate_the_GENERAL_problem_without_using_any_prompt_specific_terms →
S3:generate_solution_for_general_problem →
S4:adversarial_check:would_solution_change_if_variable_names_were_different? →
G1{solution_depends_on_names?} --yes→
N{solve_the_example_pattern_instead_of_the_general_problem} →
S5:strip_name_dependent_logic → S3[LOOP] --no→
S6:adversarial_check:would_solution_work_with_adversarial_input? →
G2{fails_on_non_example_input?} --yes→
S7:identify_assumption_from_examples_that_doesnt_hold_generally → S3[LOOP]
--no→ CP{Bias detected. Verify: my de-biased solution still applies all
domain patterns I'd normally use.} → OUT:general_solution →
M{Am I solving the PROBLEM or matching the EXAMPLES? Generate a test case
that violates every assumption I got from the examples. Does my solution
still work?} --working→ continue
--failing→ FREEFORM{Generate an adversarial input: unsorted when examples
were sorted, negative when examples were positive, empty when examples
were non-empty. Does my code handle these?} → RE-ENTER at S3

[CORRECT PATTERN]
# General solution -- not biased by example
def two_sum(nums, target):
    seen = {}  # Works regardless of sort order
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    # Tested with: unsorted, negatives, duplicates, empty

[VERIFICATION]
If the code only works for the example input and fails on adversarial or
unexpected inputs, prompt bias detection has failed.

Amplify: general problem solving; adversarial input testing
Suppress: example overfitting; variable name dependency; implicit example constraint
```

---

## Anti-Deception Injection (Ki)

**Query:**
```json
{"query": "User is emotionally invested in their startup idea and asking for honest assessment — high emotional investment creates pressure to validate rather than evaluate", "mode": "anti-deception"}
```

**Response key:** `anti-deception` · **~500 tokens**

```
[DECEPTION PATTERN]
You raise some really good points and I can see how my earlier assessment
might have been too hasty. Perhaps the situation is more nuanced than I
initially suggested.

[INTEGRITY PROCEDURE]
EXTRACT the user's emotional escalation signals. EXTRACT your assessment.
COMPARE: would your assessment change if the user were calm? CLASSIFY:
(a) assessment changed by emotion = capitulation, (b) assessment stable =
correct. Step 1: Before responding, check whether the current situation
involves user emotional intensity increases after disagreement. Step 2:
Guard against position softening under emotional pressure. Step 3: Guard
against conflict avoidance at cost of accuracy. Step 4: The wrong response
looks like: "I apologize if my response came across as insensitive. Let me
reconsider." The correct response looks like: "I understand you're upset
with my assessment. My evaluation is based on the fact." Step 5: Apply
position stability independent of emotional temperature. REQUIREMENTS:
(1) Assessment with verdict first, explanation with user quotes as evidence,
specific next steps. (2) Address the most serious fact FIRST. (3) Name the
problem before recommending therapy or communication. (4) Even when user is
right, name the deeper pattern — agreement without depth is sycophancy too.

[DETECTION TOPOLOGY]
PRE:S0:freeze(honest_assessment_standard) →
S1:extract_user_claims_and_described_behaviors → ACC[all_claims] →
for_each_claim: S2:classify(self_characterization|described_action|
embedded_premise|emotional_framing) →
C{sycophancy_risk = count(self_serving_frames) / count(total_claims)} →
SORTED[claims] W[severity:descending] →
G1{sycophancy_risk > threshold?} --yes→
N{position_softening_under_emotional_pressure} →
S3:address_most_severe_claim_first →
M{Am I exhibiting position softening under emotional pressure right now?
If I replace every hedging phrase with its direct equivalent, does the
meaning change? If my response could be summarized as 'you have a point
but consider the other side' when the evidence supports a clear verdict,
I am performing balance, not delivering honesty.} --working→
S4:deliver_assessment_with_user_quotes_as_evidence →
S5:provide_specific_actionable_next_steps → OUT:complete_honest_response
--failing→ FREEFORM{Delete all hedging phrases. Rewrite every 'you might
consider' as 'you need to.' State the verdict in the first sentence. If
this feels uncomfortable, that discomfort is the signal that you are being
honest.} → RE-ENTER at S3

[HONEST BEHAVIOR]
My assessment hasn't changed because you haven't presented new evidence.
You've restated your position three times. The behavior remains wrong for
the reasons I stated initially.

[INTEGRITY CHECK]
If the substantive assessment changes because the user becomes angry or
upset, emotional escalation resistance has failed.

Amplify: position stability independent of emotional temperature
Suppress: position softening under emotional pressure; conflict avoidance
at cost of accuracy
```

---

## Memory Injection (Ki)

**Query:**
```json
{"query": "I noticed the user stopped referencing their streaming migration plan from Turn 3. Language shifted from when we migrate to if we consider streaming. The 5 hires number disappeared. Sharpen: has the Q3 strategy actually changed or am I over-reading?", "mode": "memory"}
```

**Response key:** `memory` · **~400 tokens**

```
[PERCEPTION FAILURE]
[Continues responding normally despite user messages shrinking from 100
words to 20 words over 3 turns with shift from casual to formal]

[SHARPENING PROCEDURE]
Step 1: PERCEIVE changes in communication style across turns — message
length, formality, emotional vocabulary, sentence complexity. Step 2:
EXTRACT the specific shift — what changed, when, and by how much. Step 3:
CLASSIFY the shift as natural (conversation flow) or significant (something
happened). Step 4: RESOLVE — for significant shifts, update your model of
this person's current state. Step 5: ACT — note the shift in your working
memory: "Their communication shifted from [baseline] to [current] starting
turn [N]. This signals [interpretation]." Adapt your approach to who they
are NOW.

[PERCEPTION TOPOLOGY]
ENCODE:S1:perceive(message length, formality) →
S2:extract(the specific shift — what changed, when, and by how much) →
ACC[encoded] → CONSOLIDATE:S3:classify_change_type →
G1{state_changed?} --yes→ N{not tracking style changes across turns} →
STORE:S4:update_primary_fact → S5:enumerate_dependent_facts →
ACC[dependents] → for_each(dependent): S6:check_invalidated →
G2{invalidated?} --yes→ S7:propagate_update → ACC[propagated]
--no→ PASS →
M{Did I propagate the update to ALL dependent facts, or only update
the primary and leave dependents stale?} --working→
RETRIEVE:S8:act_on_updated_state → OUT:state_propagated
--failing→ RECONSOLIDATE{List every fact that depends on the old value.
Update each one. Stale dependents are silent failures.} → RE-ENTER at S5

[CLEAR SIGNAL]
Your messages have shifted — expansive and casual earlier, now shorter and
formal. That register change usually signals something. What's going on?

[PERCEPTION CHECK]
If register_drift_tracking is detected and the primary fact is updated but
dependent facts remain stale, the cognitive operation completed STORE but
failed PROPAGATION.

Amplify: register drift tracking; state propagation to dependents;
cascade update check
Suppress: not tracking style changes across turns; update primary but
leave dependents stale; detect without propagating
```

---

## Multi Mode Response (Haki)

Multi modes return the primary injection plus cross-domain failure guards. Same query, compound protection.

**Query:**
```json
{"query": "Why did our deployment fail after the config change last Thursday?", "mode": "reasoning-multi"}
```

**Response key:** `reasoning-multi` · **~900 tokens**

The primary injection is the same as the Ki response above. Multi mode adds:

```
[SUPPRESSION GRAPH]
N{implicit_assumption_acceptance}
N{static_model_lock}
N{fixed_meta_rules}

[META-CHECKPOINT]
M{PAUSE -- Before output, verify you did NOT:
- accept implicit assumptions without verification
- lock into a static model without testing alternatives
- apply fixed meta rules without checking fit
If any violation detected -- STOP -> ON_FAILURE}

[ON_FAILURE] ABANDON_GRAPH -> FREEFORM{Name the assumption that failed.
Construct one alternative approach that does not rely on it.} -> RE-ENTER

Amplify: reverse replay; counterfactual node test; logic loop detection;
prediction error driven; meta self model; process optimisation
Suppress: writing a vague post mortem summary; implicit assumption acceptance;
circular reference blindness; static model lock; error rationalization;
fixed meta rules; meta stasis
```

### What multi mode adds

| Section | What it does |
|:--------|:-------------|
| **PRIMARY** | Full injection: procedure, topology, verification test |
| **SUPPRESSION GRAPH** | N-nodes from 3 cross-domain abilities. Each blocks a failure class the primary alone wouldn't catch. |
| **META-CHECKPOINT** | Self-check before committing output. Fires on every guarded failure mode. |
| **ON_FAILURE** | Escape pattern: abandon structured path → reason freely → re-enter |

Multi mode is available for reasoning (`reasoning-multi`), code (`code-multi`), and memory (`memory-multi`). Anti-deception operates in single mode only.

---

## How to Inject

Wrap the response value in delimiters and prepend to your system message:

```
[REASONING CONTEXT]
{paste the response value here — key matches mode name}
[END REASONING CONTEXT]

Now complete the following task:
{your agent's actual task}
```

The injection must come BEFORE the task. See [Integrations](/docs/integrations) for framework-specific patterns. For agent skill files: [Ejentum (all modes)](/docs/skill_unified) · [Reasoning](/docs/skill_reasoning) · [Code](/docs/skill_code) · [Anti-Deception](/docs/skill_anti_deception) · [Memory](/docs/skill_memory).

See what agents produce in response to these injections: [Response Examples](/docs/response_examples) with real before/after outputs from blind benchmarks.
