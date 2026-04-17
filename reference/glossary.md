# Glossary

Key terms used throughout the Ejentum documentation. For the theory behind these concepts, see [The Method](/docs/method). For practical usage, see the [API Reference](/docs/api_reference).

## API Response Components

**Negative Gate**
The failure pattern the agent must avoid. A specific, named cognitive error that the ability is designed to prevent. Example: "Treats the first visible symptom as the root cause without tracing the causal chain deeper."

**Target Pattern**
What correct reasoning looks like for this task type. A concrete description of the reasoning behavior the ability is designed to produce.

**Reasoning Topology**
The execution structure of the ability. Defines the steps, decision gates, loops, and exit conditions the agent should follow. Example: `S1:identify → S2:trace → G1{found?} --yes→ OUT --no→ S2[LOOP]`

**Falsification Test**
A verification criterion the agent checks its output against. If the output fails this test, the reasoning procedure was not followed correctly. Example: "If the proposed fix addresses a surface symptom without tracing the causal chain, root cause analysis was skipped."

**Cognitive Payload**
The combined set of suppression vectors, amplification vectors, cognitive style, and reasoning elasticity delivered with each ability.

## Cognitive Payload Fields

**Suppression Vectors**
Failure modes to actively block during generation. These are specific reasoning shortcuts that the ability is designed to prevent. Examples: `symptom_treatment_bias`, `surface_level_stop`, `optimism_bias`, `correlation_as_causation`. In our [benchmarks](/docs/benchmarks), suppression signals produce larger behavioral improvements than amplification alone. The effect is multiplicative: each named failure mode eliminates an entire branch of incorrect outputs. Self-monitoring improved +132% primarily through suppression, not amplification.

**Amplification Vectors**
Reasoning patterns to prioritize during generation. These signals pull the model's output toward specific cognitive operations.

**Cognitive Style**
A single semantic anchor that sets the reasoning methodology for the entire generation. Examples: root cause isolation, bayesian inference, counterfactual simulation.

**Reasoning Elasticity**
Controls how tightly the agent must converge on a specific answer versus how freely it can explore. Two components:
- **Coherence target:** What the output should converge toward (e.g., calibrated probability, audit completeness)
- **Expansion factor:** How much creative latitude is allowed (conservative, adaptive, high variance, max entropy)

## Modes

**Ki Modes**
One ability per call — the highest-scoring match for your query. Available modes: `reasoning`, `anti-deception`, `code`, `memory`. Best for focused, single-domain tasks.

**Haki Modes**
Primary ability plus cross-domain failure guards. Available modes: `reasoning-multi`, `code-multi`, `memory-multi`. The multi injection contains:
- **Primary:** Full ability — procedure, topology, verification test
- **Failure Guards:** Specific reasoning failures to block, extracted from 3 additional abilities in different domains
- **Self-Check:** Dynamic verification that fires before the agent commits to output
- **Escape Pattern:** When structured reasoning fails, the agent can break out, think freely, and re-enter

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
The core methodology. Unlike RAG (Retrieval-Augmented Generation) which retrieves information, RA2R retrieves reasoning abilities. The agent receives structured cognitive injections that govern HOW it thinks, not WHAT it knows.

**Cognitive Scaffold**
The complete reasoning structure delivered per ability. Includes negative gate, reasoning procedure, topology, target pattern, suppression/amplification vectors, and falsification test.

**Compound Suppression**
In multi modes, failure-blocking signals from the primary and 3 cross-domain abilities are merged. Each additional ability contributes specific failure guards that catch reasoning breakdowns the primary alone would miss.

**Ability**
A single engineered cognitive operation in the [679-ability catalog](/abilities). Each ability is a 20-field protocol including suppression vectors, reasoning topology, falsification test, and synergy edges. See [Concepts](/docs/concepts) for the full anatomy.
