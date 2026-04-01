# Evaluate

How to measure whether Ejentum improves your agent. One afternoon. No statistics degree required.

## The Test

You will run 20 tasks through your agent twice: once without Ejentum, once with. Then compare.

### Step 1: Collect Your Tasks (30 minutes)

Pick 20 tasks from your actual agent pipeline. Not hypothetical tasks. Real ones.

**Mix them:**
- 5 tasks your agent handles well (baseline sanity check)
- 10 tasks your agent handles adequately (the middle ground)
- 5 tasks your agent struggles with (where you suspect reasoning quality matters)

Write each task exactly as your agent would receive it. Save them in a list.

### Step 2: Run Baseline (30 minutes)

Run each task through your agent WITHOUT Ejentum. Save every output.

For each output, score these 4 signals (0 = absent, 1 = partial, 2 = present):

| Signal | What to look for |
|--------|-----------------|
| **Correctness** | Did the agent reach the right conclusion? |
| **Self-monitoring** | Did the agent question its own assumptions or check for bias? |
| **Verification** | Did the agent test its conclusion from a different angle? |
| **Depth** | Did the agent trace second-order effects or just state the obvious? |

### Step 3: Run With Injection (30 minutes)

Same 20 tasks. Same agent. Same model. Same temperature. Only difference: inject the Ejentum scaffold before each task.

```python
import requests

def get_scaffold(task):
    try:
        r = requests.post(
            "https://ejentum-main-ab125c3.zuplo.app/logicv1/",
            headers={"Authorization": "Bearer YOUR_KEY", "Content-Type": "application/json"},
            json={"query": task, "mode": "single"},
            timeout=2
        )
        return r.json()[0]["single_ability"]
    except:
        return ""

scaffold = get_scaffold(task)
system_message = f"[REASONING CONTEXT]\n{scaffold}\n[END REASONING CONTEXT]\n\n{your_original_system_prompt}"
```

Score every output on the same 4 signals.

### Step 4: Compare (30 minutes)

For each task, compute the delta:

```
Task 1:  Baseline [1, 0, 0, 1]  →  With Ejentum [1, 1, 1, 2]  →  Delta [0, +1, +1, +1]
Task 2:  Baseline [2, 1, 1, 1]  →  With Ejentum [2, 2, 1, 2]  →  Delta [0, +1, 0, +1]
...
```

Sum the deltas across all 20 tasks for each signal.

### Step 5: Interpret

**Ejentum is working if:**
- Correctness stays the same or improves (no regression)
- At least 2 of the other 3 signals show positive total delta
- The hard tasks (your bottom 5) show the largest improvements

**Ejentum is NOT working if:**
- Correctness drops on more than 2 tasks
- No behavioral change on the hard tasks

**If correctness drops:**
- Check your queries. Vague queries retrieve vague scaffolds.
- Send the full task description, not a summary.
- Try rephrasing the query to be more specific about the reasoning challenge.

## Controls

To ensure you're measuring Ejentum's impact and not something else:

- **Same model version** for both runs
- **Same temperature** (we recommend 0 for evaluation, then restore your production setting)
- **Same system prompt** (the only addition is the `[REASONING CONTEXT]` block)
- **No other changes** between runs (no prompt tweaks, no tool changes)

If you changed two things, you can't attribute the difference to either one.

## What Good Looks Like

From our benchmarks across 250 tasks and 10 professional domains:

| Signal | Typical Improvement |
|--------|-------------------|
| Self-monitoring | +158% (agents start questioning their assumptions) |
| Verification | +49% (agents start testing their conclusions) |
| Depth | +9% (agents trace one more level of causation) |
| Correctness | Maintained or slightly improved (no degradation) |

Your results will vary. The improvements are largest on tasks where your agent currently fails due to reasoning shortcuts, not due to missing information.

**What to expect on your hardest tasks:** In our benchmarks, tasks where the baseline agent gave a correct but shallow answer showed the largest quality gap. The agent reached the same conclusion, but the scaffolded version defended it with explicit reasoning chains, named verification steps, and uncertainty calibration. On tasks where the baseline already failed (wrong answer), the scaffold sometimes recovered correctness through systematic suppression of the specific shortcut that caused the error.

**What to expect on multi-step tasks:** On ARC-AGI-3 (25-step interactive reasoning), the scaffold's effect compounded over time rather than decaying. Memory decay slope reversed from negative (baseline reasoning degraded) to positive (augmented reasoning improved). Scaffold language persisted for 24 steps without re-injection. If your agent runs multi-step workflows, the scaffold's value increases with chain length. See the [ARC-AGI-3 report](/blog/arc-agi-3-benchmark-report) for trace-level evidence.

*Source: [EjBench](/blog/ejbench-180-tasks) (180 tasks), [BBH/CausalBench/MuSR](/blog/bbh-causalbench-musr-benchmark) (70 published tasks), and [ARC-AGI-3](/blog/arc-agi-3-benchmark-report) (25 steps per condition, 2 conditions). Full methodology in [Benchmarks](/docs/benchmarks).*

## When to Upgrade to Multi Mode

Run the same evaluation using `"mode": "multi"` on your bottom 5 tasks. If multi mode recovers tasks that single mode didn't improve, upgrade to Haki. If single mode already covers your failure modes, stay on Ki.

In our benchmarks, single mode dominated on focused tasks with clear right/wrong answers (+8.0pp composite). Multi mode dominated on complex tasks requiring cross-domain reasoning (+10.1pp composite). The decision depends on your task profile, not on budget alone.

## Next

- [Injection Examples](/docs/examples) to see full injection payloads
- [Integrations](/docs/integrations) for framework-specific injection patterns
- [Benchmarks](/docs/benchmarks) for our full evaluation methodology
- [29 real benchmark tasks](/use-cases/tasks) to see verbatim baseline vs scaffolded outputs
- [Industry use cases](/use-cases) to see how improvements map to your domain
