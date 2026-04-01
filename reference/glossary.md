# Glossary

Key terms used throughout the Ejentum documentation.

## API Response Components

**Negative Gate**
The failure pattern the agent must avoid. A specific, named cognitive error that the ability is designed to prevent. Example: "Treats the first visible symptom as the root cause without tracing the causal chain deeper."

**Target Pattern**
What correct reasoning looks like for this task type. A concrete description of the reasoning behavior the scaffold is designed to produce.

**Reasoning Topology**
The execution structure of the ability. Defines the steps, decision gates, loops, and exit conditions the agent should follow. Example: `S1:identify → S2:trace → G1{found?} --yes→ OUT --no→ S2[LOOP]`

**Falsification Test**
A verification criterion the agent checks its output against. If the output fails this test, the reasoning procedure was not followed correctly. Example: "If the proposed fix addresses a surface symptom without tracing the causal chain, root cause analysis was skipped."

**Cognitive Payload**
The combined set of suppression vectors, amplification vectors, cognitive style, and reasoning elasticity delivered with each ability.

## Cognitive Payload Fields

**Suppression Vectors**
Failure modes to actively block during generation. These are specific reasoning shortcuts that the ability is designed to prevent. Examples: `symptom_treatment_bias`, `surface_level_stop`, `optimism_bias`, `correlation_as_causation`. In our benchmarks, suppression signals produce larger behavioral improvements than amplification alone. The effect is multiplicative: each named failure mode eliminates an entire branch of incorrect outputs. Self-monitoring improved +132% primarily through suppression, not amplification.

**Amplification Vectors**
Reasoning patterns to prioritize during generation. These signals pull the model's output toward specific cognitive operations.

**Cognitive Style**
A single semantic anchor that sets the reasoning methodology for the entire generation. Examples: root cause isolation, bayesian inference, counterfactual simulation.

**Reasoning Elasticity**
Controls how tightly the agent must converge on a specific answer versus how freely it can explore. Two components:
- **Coherence target:** What the output should converge toward (e.g., calibrated probability, audit completeness)
- **Expansion factor:** How much creative latitude is allowed (conservative, adaptive, high variance, max entropy)

## Modes

**Single Mode — Ki (single ability)**
Returns one ability per call — one human-like engineered cognitive operation, structured and optimized for agentic inference. The highest-scoring scaffold for the given query. Best for focused, single-domain tasks.

**Multi Mode — Haki (multi ability)**
Returns four synergized abilities per call — four cognitive operations composed into a single compound scaffold with merged suppression and amplification vectors. The four roles:
- **Primary:** Sets the reasoning direction
- **Dependency:** Grounds the primary in prerequisites
- **Amplifier:** Deepens the analysis
- **Alternative:** Challenges the conclusion

## Reasoning Dimensions

**Causal**
Addresses direction-of-causation errors. Prevents the agent from treating correlation as causation.

**Temporal**
Addresses sequence and duration errors. Prevents the agent from confusing past and future or confabulating timelines.

**Spatial**
Addresses boundary and topology errors. Prevents the agent from violating physical or structural constraints.

**Simulation**
Addresses counterfactual and consequence modeling errors. Prevents the agent from ignoring downstream effects.

**Abstraction**
Addresses category and generalization errors. Prevents the agent from conflating metaphor with mechanism.

**Metacognition**
Addresses self-awareness failures. Prevents the agent from continuing to reason incorrectly without recognizing its own degradation.

## Other Terms

**RA2R (Reasoning Ability-Augmented Retrieval)**
The core methodology. Unlike RAG (Retrieval-Augmented Generation) which retrieves information, RA2R retrieves reasoning abilities. The agent receives structured cognitive scaffolds that govern HOW it thinks, not WHAT it knows.

**Cognitive Scaffold**
The complete reasoning structure delivered per ability. Includes negative gate, reasoning procedure, topology, target pattern, suppression/amplification vectors, and falsification test.

**Compound Suppression**
In multi-ability mode, suppression vectors from all four abilities are merged into a single set. This catches failure modes that any individual ability would miss.
