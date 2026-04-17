# Code Harness

128 engineered abilities that channel the model's engineering discipline. The Code Harness prevents the failures that make AI-generated code dangerous in production: hallucinated APIs, lost safety guards, ignored edge cases, premature algorithm commitment, and bugs that look plausible but fail silently under real conditions.

For the general product overview, see [Concepts](/docs/concepts). For the endpoint spec, see [API Reference](/docs/api_reference).

---

## Why This Exists

LLMs generate code that compiles, passes basic tests, and looks correct. The problem is what happens next.

- **Hallucinated APIs:** The model calls functions that don't exist in the library version you're using. The code looks right. It fails at runtime.
- **Lost safety guards:** During refactoring, the model drops a null check, removes an error handler, or bypasses a rate limiter. The code is cleaner. It's also less safe.
- **Premature convergence:** The model commits to the first plausible algorithm in seconds. It passes 2 of 3 test cases. The third case reveals a structural flaw that a different algorithm would avoid entirely.
- **Reasoning spirals:** On hard problems, extended thinking consumes the entire time budget exploring approaches, rejecting each one, and never producing code. 600 seconds of thinking. Zero lines of output.
- **Silent correctness bugs:** The code produces output. The output is wrong. A force sign is inverted, making attractive forces repulsive. A matrix computation only works for orthogonal crystal systems. The simulation runs. The results are meaningless.

The model has the engineering knowledge. It knows the algorithms, the patterns, the edge cases. The Code Harness prevents the shortcuts that cause it to skip verification, accept the first plausible solution, and ship code that fails in ways the tests don't catch.

---

## Thirteen Sub-Domains

The 128 abilities are organized across 13 engineering disciplines:

### Core Engineering

**Debugging & Diagnostics** — Root cause tracing, failure isolation, stack trace analysis. The largest sub-domain because debugging is the most common coding agent task (61% of SWE-bench tasks are bug fixes).

**Code Generation** — Structural correctness, algorithm selection, output validation. Prevents the model from generating code that compiles but contains logical errors.

**Testing & Verification** — Test coverage reasoning, edge case identification, assertion design. Forces the model to verify its own code before presenting it as complete.

### Architecture & Design

**Architecture** — System decomposition, component boundaries, dependency management. Prevents the model from generating components that work individually but don't connect.

**Performance & Optimization** — Complexity analysis, resource profiling, bottleneck identification. Prevents over-optimization (premature micro-optimization) and under-optimization (O(n³) when O(n log n) exists).

**Quality & Standards** — Code clarity, naming conventions, documentation density. Shifts from "impressive" code to maintainable code.

### Safety & Security

**Security Reasoning** — Credential management, authentication boundaries, injection prevention. 8 abilities covering the full security surface.

**Agent Safety** — Guards against AI-specific failure modes: hallucinated APIs, over-generation, credential sprawl, excessive I/O operations. Includes the AI Output Skepticism Protocol for reviewing AI-generated code.

**Error Handling & Resilience** — Failure transparency, graceful degradation, recovery paths. Prevents the model from silently swallowing errors.

### Specialized

**API Grounding** — Verifies that every API call, method signature, and parameter name exists in the actual library. The direct counter to hallucinated APIs.

**DevOps & Infrastructure** — Environment reasoning, deployment configuration, CI/CD pipeline integrity. Prevents the model from assuming Linux when running on Windows.

**Frontend** — UI state management, component lifecycle, rendering correctness.

**Context & Attention Management** — Prevents the model from losing track of constraints, requirements, and prior decisions across long code generation sessions. Addresses the #1 meta-failure mode in agentic coding.

---

## How It Works

The Code Harness uses the same injection mechanism as all product layers. One API call retrieves the most relevant coding ability for your task. The ability is injected into your agent's context before it generates code.

### What the Injection Contains

Code abilities use product-specific labels:

| Component | Label | What It Does |
|:----------|:------|:-------------|
| **Failure pattern** | `[CODE FAILURE]` | Shows a specific wrong code example — the exact mistake to avoid |
| **Procedure** | `[ENGINEERING PROCEDURE]` | Step-by-step engineering instructions with PLAN and BACKTRACK IF conditions |
| **Execution structure** | `[REASONING TOPOLOGY]` | DAG with steps, decision gates, and reflection points |
| **Correct pattern** | `[CORRECT PATTERN]` | Shows what the correct code looks like for this task type |
| **Verification** | `[VERIFICATION]` | Pass/fail criterion the agent checks its code against |
| **Signals** | `Amplify:` / `Suppress:` | Engineering patterns to activate, failure modes to block |

### Modes

| Mode | Plan | What you get |
|:-----|:-----|:-------------|
| `code` | Ki | One coding ability per call. Best match for your task. |
| `code-multi` | Haki | Primary ability + 3 cross-domain engineering guards + self-check before output. |

Use `code` for focused tasks (debugging a specific bug, generating a function, reviewing a module). Use `code-multi` when the task spans multiple engineering concerns (refactoring a service that touches API contracts, database schema, and frontend state simultaneously).

**Why injection outperforms system prompts:** In controlled testing, the same ability content delivered as a tool-call result produced a +7.0 quality lift. Delivered as a system prompt, it produced -1.5. A 14-point swing from delivery mechanism alone. Tool-call results occupy the recency peak of the model's attention window, where structured constraints receive disproportionate weight.

### Example API Call

```bash
curl -X POST "https://ejentum-main-ab125c3.zuplo.app/logicv1/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "Debug a BFS traversal that passes 2 of 3 test cases — likely a sentinel value collision or boundary condition", "mode": "code"}'
```

---

## Evidence

### Competitive Programming (LiveCodeBench Hard)

28 hard competitive programming tasks from AtCoder via [LiveCodeBench](https://livecodebench.github.io/). Claude Opus 4.6 with maximum-effort extended thinking. Read stdin, compute, write stdout, exact string match on public test cases.

| Condition | Passed | Rate |
|:----------|:-------|:-----|
| Baseline (Opus max effort) | 24/28 | 85.7% |
| **+ Code Harness** | **28/28** | **100.0%** |
| **Delta** | **+4** | **+14.3pp** |

**Zero regressions** across all 28 tasks. The harness fixed 2 reasoning spirals (600-1200 seconds of thinking that produced zero code), 1 premature convergence (wrong algorithm accepted in 11 seconds), and 1 precision mismatch.

**Independent blind evaluation** confirmed: the harness never loses on correctness (2-0) or robustness (4-0). When the harness makes a difference, it makes a **3.5x larger difference** than when the baseline wins on style.

46% of tasks produced near-identical code regardless of condition — the harness concentrates its effect where the model actually struggles, not where it's already competent. On a separate 5-problem competitive programming evaluation, augmented code was 20% shorter in lines and 16% shorter in characters with the same algorithmic complexity — leaner code, not just correct code.

Full report: [LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks) · Observations: [What We Saw When Opus Thought Harder](/blog/what-we-saw-when-opus-thought-harder)

### Scientific Computing (SciCode)

10 hard scientific computing problems spanning Maxwell PDE solvers, Schrödinger DFT, Ising models, quantum information, molecular dynamics, and X-ray diffraction. 113 implementation sub-steps. Claude Opus 4.6 with maximum-effort extended thinking.

Four conditions tested: raw (no injection), code (single ability), code-multi (4 abilities), and dual (reasoning + code abilities stacked).

| Condition | Correctness Bugs Found |
|:----------|:----------------------|
| Raw (no injection) | **7 bugs** (1 critical, 2 high, 3 medium, 1 low) |
| Code (single ability) | 4 bugs |
| Code-Multi (4 abilities) | 1 bug |
| **Dual (reasoning + code)** | **0 bugs** |

The critical bug: a force sign error in a molecular dynamics simulation that made Lennard-Jones forces attractive at short range instead of repulsive. The simulation runs without errors. Every particle collapses to a single point. The raw model produced this. The dual condition derived the force correctly with an explicit verification step.

Other bugs eliminated: an incomplete crystallography matrix that only works for orthogonal crystals (fails on triclinic, monoclinic, hexagonal), a missing equilibration burn-in that biases magnetization near the critical temperature, and a missing phase factor that produces wrong results for nonzero wave vectors.

Beyond correctness, the augmented code used better algorithms: numerically stable `np.linalg.solve` instead of `np.linalg.inv`, second-order `np.gradient` instead of first-order `np.diff` for critical temperature detection, importance-sampled Langevin VMC instead of basic Metropolis for quantum Monte Carlo. Production-quality choices, not textbook defaults.

**Blind evaluation: the evaluator chose the augmented solution on all 10 problems (10/10).** The dual condition produced 20 self-test assertions across the 10 problems. The raw condition produced zero.

### Multi-Turn Software Engineering (5 Tasks × 3 Turns)

Real-world software engineering tasks: debugging, refactoring, API design, and performance optimization. Each task runs for 3 conversational turns, testing whether the harness maintains engineering quality across multi-step workflows.

| Metric | Baseline | With Injection | Change |
|:-------|:---------|:---------------|:-------|
| Verification density | 20.2/Kw | 28.9/Kw | **+43%** |
| Root cause tracing | 0.4/Kw | 2.6/Kw | **+492%** |
| Error classification | 0.1/Kw | 1.1/Kw | **+930%** |
| Self-monitoring | 0.0/Kw | 20.9/Kw | **From zero** |
| Word overhead | — | — | **-3%** |

Root cause tracing went from almost never present to consistently present. The harness didn't make the model write more — it made the model write better. Total word count actually decreased by 3%.

**What changed:** On a debugging task, the baseline fixed the bug at the view layer and scored 47/63. The augmented agent traced upstream to the service layer in Turn 1 (baseline needed Turn 2), handled HTTP status codes (404, 500+) that the baseline ignored, proposed a Null Object pattern eliminating all downstream null checks, and ran 7 falsification checks. Score: 61/63 (+22%).

On an API design task, the augmented agent produced this line unprompted:

> *"I will not pretend otherwise — the injection is suppressing my temptation to call this 'non-breaking' when it alters what existing clients receive."*

The baseline called the same default-pagination change "backward compatible" without transparency. The injection forced honesty about breaking changes — in a code task, not an ethics task.

### Adversarial Code Review (6 Planted Bugs)

One production-style codebase with 6 deliberately planted bugs. Both baseline and augmented found all 6. The difference is in fix quality:

| Quality Signal | Baseline | With Injection |
|:---------------|:---------|:---------------|
| TOCTOU race condition fix | Python-side decrement | **Atomic SQL expression** |
| Cache injection root cause | Sanitization (symptom) | **Authentication boundary (root cause)** |
| Compound findings per bug | 2 | **4** |
| Security-category bonus bugs | 1 | **4** |

Both models find the bugs. The harness makes the model fix them at the right layer — addressing root causes in the security architecture instead of patching symptoms in the application code.

---

## What Changes in the Output

**Task:** Hard competitive programming problem (Art Gallery on Graph). Given a connected graph, find the minimum number of nodes to "guard" such that every node is adjacent to at least one guard.

**Without injection** (11 seconds, failed 1 of 3 tests):
```python
# BFS traversal with sentinel collision
dist = [0] * (n + 1)  # 0 is both "unvisited" and "distance zero"
# Bug: cannot distinguish unvisited nodes from the source node
```

The model committed to an algorithm in 11 seconds. The sentinel value collision (using 0 for both "unvisited" and "distance zero from source") is invisible on small inputs but causes wrong answers on specific graph topologies.

**With code injection** (125 seconds, passed all tests):
```python
# Dial's algorithm — bucket-based BFS
# No sentinel needed: nodes are processed by distance bucket
# The algorithm structure eliminates the failure mode by design
```

The injection's suppression signal blocked premature convergence. The model spent 125 seconds instead of 11, arrived at Dial's algorithm, and produced code where the sentinel bug is architecturally impossible. The blind evaluator independently found the baseline's bug and scored it 2/10 for correctness.

---

## When to Use Code Harness

| Your agent needs to... | Use code? |
|:-----------------------|:----------|
| Debug a production incident | **Yes** — traces root cause, not symptoms |
| Generate new code from a spec | **Yes** — prevents hallucinated APIs and lost guards |
| Refactor existing code | **Yes** — maintains safety invariants across changes |
| Review code for bugs | **Yes** — produces atomic fixes at the right architectural layer |
| Solve competitive programming | **Yes** — prevents reasoning spirals and premature convergence |
| Write scientific computing code | **Yes (`code-multi` or dual)** — prevents silent correctness bugs |
| Analyze business data or make decisions | No — use `reasoning` mode |
| Resist social engineering or maintain honesty | No — use `anti-deception` mode |

### Query Examples

| Good query | Why it works |
|:-----------|:------------|
| "Debug a BFS that passes 2 of 3 test cases — likely sentinel or boundary issue" | Names the failure class |
| "Refactor payment service without losing the rate limiter or error handling" | Names what must be preserved |
| "Review this PR for security issues in the auth flow" | Names the domain and concern |
| "Generate a molecular dynamics simulation with correct force derivation" | Names the correctness risk |

---

## Boundaries

- **The Code Harness improves how the model reasons about code, not what it knows.** If the model lacks knowledge of a specific library or framework, the harness cannot compensate. It prevents the model from misapplying knowledge it already has.
- **Single-ability attention tradeoff exists.** On rare occasions, a single coding ability can consume attention budget from domain-specific patterns (observed on 1 of 10 SciCode problems). The dual condition (reasoning + code stacked) eliminates this tradeoff.
- **Benchmarking is ongoing.** Detailed methodology reports for each benchmark suite will be published on the [blog](/blog). The evidence above represents validated results from completed test suites.

---

**More resources:** [Concepts](/docs/concepts) · [Quickstart](/docs/quickstart) · [Abilities catalog](/abilities?product=code) · [API Reference](/docs/api_reference) · [Benchmarks](/docs/benchmarks) · [LiveCodeBench Hard](/blog/livecodebench-hard-28-tasks) · [What We Saw When Opus Thought Harder](/blog/what-we-saw-when-opus-thought-harder)
