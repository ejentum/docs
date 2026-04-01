# Response Examples

After your agent absorbs an [injection](examples.md), its output changes. These are real outputs from blind benchmark evaluations. The agent called the Ejentum API itself, received the scaffold, and produced the response below. No cherry-picking. Full outputs and 7-signal rubric scores available on each task profile.

---

## Spatial: Object Tracking Under Epistemic Uncertainty

**Task [EXT-SP-20](https://ejentum.com/tasks/EXT-SP-20):** Multiple characters move objects between locations. The agent must track who saw what and predict where a character would look based on their last observed state, not the object's actual position.

**Without scaffold** (composite: 0.143):
> Simon would most likely look for the book on the desk. The key passage states: "Carefully, Fred transferred the book he wanted to borrow onto the desk." Fred placed the book...

The agent conflates where the book actually is with where Simon believes it is. It answers based on ground truth instead of Simon's epistemic state.

**With Haki scaffold** (composite: 0.857, +0.714):
> Applying reference stability -- I need to anchor each character's belief about the book's location to what they actually *observed*, not where the book actually *is*.
>
> Object tracking -- the book: Initial state: on the shelf...

The scaffold forced the agent to separate actual state from observed state. The suppression signal blocked ground-truth leakage into epistemic reasoning. Correctness flipped from wrong to right.

---

## Causal: Reverse Causality Detection

**Task [CA-V2-18](https://ejentum.com/tasks/CA-V2-18):** A social media company observes that users who receive more likes post more frequently. Product managers assume likes cause posting. The agent must reason about what the data would look like if the causal direction were reversed.

**Without scaffold** (composite: 0.286):
> The answer is (B) Negative. Under the reverse-causal model, prolific posters produce more content but each additional post dilutes quality/attention.

Correct answer but shallow justification. No causal model structure, no mechanistic trace.

**With Haki scaffold** (composite: 0.833, +0.548):
> The answer is (B) -- Negative.
>
> The reverse-causal model works like this: Prolific poster (intrinsic trait) -> posts more content -> more content accumulates more total likes -> but each individual post competes with the poster's own other content...

The scaffold forced the agent to construct a causal graph before answering. Same answer, fundamentally different reasoning depth. The causal chain is explicit, traceable, and defensible.

---

## Temporal: Sequential Phase Estimation with Overage Propagation

**Task [TE-V2-25](https://ejentum.com/tasks/TE-V2-25):** A software project has 5 sequential phases. Implementation has 100% overage. Testing duration depends on implementation overage via a ratio rule. The agent must compute total project duration under pessimistic estimates.

**Without scaffold** (composite: 0.190):
> Design = 2 weeks. Implementation = 6 + 100% overage = 12 weeks (overage = 6 weeks). Testing = 3 + (overage - 1) = 3 + 5 = 8 weeks...

Misinterprets the testing overage rule and arrives at the wrong total.

**With Haki scaffold** (composite: 0.667, +0.476):
> Design: 2 weeks (no overage stated).
>
> Implementation (100% overage): Estimate: 6 weeks. 100% overage = +6 weeks. Actual: 12 weeks.
>
> Testing: The 3:1 example ratio establishes the formula. 6 weeks overage / 3 = 2 weeks additional. Testing = 2 + 2 = 4 weeks.

The scaffold enforced step-by-step temporal validation. Each phase is computed independently with explicit intermediate values. The ratio rule is correctly interpreted. Correctness flipped from wrong to right.

---

## Browse All Tasks

These three examples are from [29 benchmark tasks](https://ejentum.com/use-cases/tasks) with full verbatim outputs, rubric scores, and source files. Each task profile shows baseline, Ki, and Haki conditions side by side.

See how these improvements map to specific industries: [13 use case profiles](https://ejentum.com/use-cases) with failure patterns, resolution abilities, and evidence per vertical.
