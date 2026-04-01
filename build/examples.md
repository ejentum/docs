# Injection Examples

Real, complete injection payloads. Not truncated. These are what your agent receives and absorbs before reasoning.

## Response Structure

Every response contains these sections:

| Section | What It Does |
|---------|-------------|
| **[NEGATIVE GATE]** | A concrete scenario of the failure pattern to avoid. The agent recognizes this as "what NOT to do." |
| **Step 1-N** | The reasoning procedure. Explicit instructions the agent follows sequentially. |
| **[REASONING TOPOLOGY]** | Execution structure as a directed graph: steps, decision gates, loops, exit conditions. |
| **[TARGET PATTERN]** | What correct reasoning looks like. A concrete example of the expected output quality. |
| **[COGNITIVE PAYLOAD]** | Amplification signals (what to prioritize), suppression signals (what to block), cognitive style, and elasticity. |
| **[FALSIFICATION TEST]** | A verification criterion. If the output fails this test, the scaffold was not followed. |

---

## Query

```json
{
  "query": "Why did our deployment fail after the config change last Thursday?",
  "mode": "single"
}
```

---

## Single Mode Response (Ki)

One cognitive scaffold. ~2,500 characters (~500 tokens). The highest-scoring ability for this query.

```
[REASONING INJECTION]
Phase: MONITORING_LOOP | Confidence: ASSERT | Trigger: QUALITY_FAIL

[NEGATIVE GATE]
The checkout service crashed and we've documented the incident — cascading
errors across three microservices, root cause still unknown. Summary: payment
processing failure leading to order corruption. Moving on to the next sprint.

Step 1: Identify the failed output and trace the causal chain backward from
the failure point to the inputs. Step 2: Isolate each reasoning step and check
whether its output correctly follows from its inputs. Step 3: Flag the first
step where output diverges from valid inference and classify the error as
logical, factual, or procedural. Step 4: Verify that the identified error is
the root cause by simulating a corrected version and propagating it forward.
If the fix fails, probe deeper. Step 5: Extract a corrective rule to prevent
recurrence. Step 6: Never accept a surface-level explanation for a failure.
If replay does not isolate the bug, decompose the chain further. Amplify
reverse replay in your analysis. Apply meta debugging throughout this process.

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

[COGNITIVE PAYLOAD]
Amplify: reverse_replay; counterfactual_node_test; patch_generation
Suppress: writing_a_vague_post_mortem_summary_instead_of_isolating_the_
specific_root_cause_node; replaying_the_chain_only_forward_and_missing_
upstream_injector_failures; applying_a_fix_patch_without_simulating_the_
corrected_chain_to_verify_no_new_failures
Cognitive Style: meta_debugging
Elasticity: coherence=root_cause_found, expansion=conservative

[FALSIFICATION TEST]
If an error's origin is not traced by replaying the reasoning chain step by
step to the divergence point, causal replay debugging was skipped.
```

See the [Response Structure](#response-structure) table above for what each section does.

---

## Multi Mode Response (Haki)

Four synergized scaffolds. 4,379 characters. Same query, compound reasoning.

```
[PRIMARY]
Phase: MONITORING_LOOP | Confidence: ASSERT | Trigger: QUALITY_FAIL

[NEGATIVE GATE]
The checkout service crashed and we've documented the incident — cascading
errors across three microservices, root cause still unknown...

(Same primary scaffold as single mode — full negative gate, reasoning
procedure, topology, target pattern, and falsification test.)

[DEPENDENCY]
[NEGATIVE GATE] The code crashes because the assumption of valid input was
never questioned, leading to a dead path.
Step 1: Scan the reasoning chain as executable logic and identify each step's
input, operation, and output. Step 2: Test each step's output against its
predecessor for logical consistency.
[FALSIFICATION TEST] If no reasoning step has been tested for logical
consistency with its predecessor, logic debugging was not performed.

[AMPLIFIER]
[NEGATIVE GATE] The sales forecast remains unchanged despite market shifts;
ignoring surprise maintains model stasis.
Step 1: Before processing new information, state what you predict the data to
look like. Enumerate key variables and expected values. Step 2: Process actual
information; compare each variable against prediction and flag every divergence.
[FALSIFICATION TEST] If prediction errors are ignored rather than being used to
update the internal model, predictive coding error minimization was not active.

[ALTERNATIVE]
[NEGATIVE GATE] The project management process is locked into a fixed
abstraction, ignoring any need for change or adaptation.
Step 1: Build a meta self-model — identify which abstraction strategy you use
and detect state change signaling it no longer fits. Step 2: Check for process
lock — repeated application of one framework without examining alternatives
indicates meta stasis.
[FALSIFICATION TEST] If the same abstraction framework is applied without
examining whether a different approach would fit the problem, meta-reflection
was skipped.

[MERGED VECTORS]
Amplify: reverse replay; counterfactual node test; patch generation; logic
loop detection; dead end identification; exception handling audit; prediction
error driven; minimal model update; precision weighting; meta self model;
process optimisation
Suppress: writing a vague post mortem summary instead of isolating the specific
root cause node; replaying the chain only forward and missing upstream injector
failures; applying a fix patch without simulating the corrected chain to verify
no new failures; implicit assumption acceptance; circular reference blindness;
static model lock; error rationalization; fixed meta rules; meta stasis
```

### What multi mode adds

| Role | What it does | Why it matters |
|------|-------------|----------------|
| **PRIMARY** | Sets the reasoning direction (same as single mode) | The core scaffold |
| **DEPENDENCY** | Grounds the primary in prerequisites (logic consistency) | Prevents the primary from building on unverified assumptions |
| **AMPLIFIER** | Deepens analysis (prediction error detection) | Forces the agent to notice when reality diverges from expectation |
| **ALTERNATIVE** | Challenges the conclusion (meta-reflection) | Prevents lock-in to one framework when the situation has changed |
| **MERGED VECTORS** | Combined suppression/amplification from all four | 11 amplification + 9 suppression signals, broader coverage than any single scaffold |

---

## How to inject

Wrap the response value in delimiters and prepend to your system message:

```
[REASONING CONTEXT]
{paste single_ability or multi_ability value here}
[END REASONING CONTEXT]

Now complete the following task:
{your agent's actual task}
```

The scaffold must come BEFORE the task. See [Integrations](/docs/integrations) for framework-specific patterns.

See what agents produce in response to these injections: [Response Examples](/docs/response_examples) with real before/after outputs from blind benchmarks.
