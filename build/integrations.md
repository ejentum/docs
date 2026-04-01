# Integrations

Ejentum is a single POST endpoint returning JSON. Any system that can make an HTTP request and inject text into a prompt works with the Logic API. No SDK required. No dependencies.

**Choose your path:** [No code (n8n)](#n8n) · [Python (LangChain)](#langchain--langgraph) · [Multi-agent (CrewAI)](#crewai) · [Claude (Agent SDK)](#claude-code--agent-sdk) · [IDE (Cursor/Windsurf)](#agentic-ides-cursor-windsurf-antigravity-codex) · [Make.com](#makecom) · [Any framework](#universal-pattern)

## The Injection Principle

Where you inject the reasoning scaffold matters as much as what you inject.

**1. Inject BEFORE the task, not after.**
The scaffold must be the first structured content the model processes. LLMs attend most strongly to content at the beginning of context.

**2. Inject into the SYSTEM message, not the user message.**
The system message sets the model's operational mode. Injecting into the user message treats the scaffold as data to reason about, not as a constraint to follow.

**3. Keep the injection SEPARATE from your instructions.**
The `[REASONING CONTEXT]...[END REASONING CONTEXT]` delimiters create a distinct attention block. Do not merge the scaffold into natural language instructions.

**4. Re-inject per turn in multi-turn agents.**
Reasoning scaffolds degrade over long contexts. Call the API for each new task step and inject fresh. Our research shows that scaffolds act as persistent attention anchors, but they lose effectiveness as task-specific tokens accumulate over extended chains. Re-injection maintains the effect.

---

## n8n

The fastest path to testing Ejentum without writing code.

### Pattern

Add an **AI Agent** node to your workflow. Connect an **HTTP Request Tool** node to the agent's **Tools** input. The agent calls the Ejentum API as a tool during execution.

### Steps

1. Add an **AI Agent** node (Tools Agent type)
2. Add an **HTTP Request Tool** node and connect it to the agent's Tools input
3. Configure the HTTP Request Tool:
   - Method: POST
   - URL: `https://ejentum-main-ab125c3.zuplo.app/logicv1/`
   - Authentication: Header Auth with your API key
   - Body: `{"query": "{task_description}", "mode": "single"}`
4. The agent receives the scaffold in the tool response and uses it to guide its reasoning

The API returns a pre-rendered string. No field assembly needed.

---

## LangChain / LangGraph

### LCEL Chain Pattern

```python
import requests
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

EJENTUM_URL = "https://ejentum-main-ab125c3.zuplo.app/logicv1/"
EJENTUM_KEY = "YOUR_API_KEY"

def get_reasoning(task: str, mode: str = "single") -> str:
    try:
        r = requests.post(
            EJENTUM_URL,
            headers={"Authorization": f"Bearer {EJENTUM_KEY}", "Content-Type": "application/json"},
            json={"query": task, "mode": mode},
            timeout=2
        )
        r.raise_for_status()
        injection = r.json()[0][f"{mode}_ability"]
        return f"[REASONING CONTEXT]\n{injection}\n[END REASONING CONTEXT]"
    except Exception:
        return ""

prompt = ChatPromptTemplate.from_messages([
    ("system", "{reasoning}\n\nYou are a senior analyst."),
    ("human", "{task}")
])

chain = (
    RunnablePassthrough.assign(reasoning=lambda x: get_reasoning(x["task"]))
    | prompt
    | ChatOpenAI(model="gpt-4o")
)

result = chain.invoke({"task": "Why did our supply chain costs spike in Q3?"})
```

### As a LangChain Tool

```python
from langchain.tools import tool

@tool
def inject_reasoning(query: str) -> str:
    """Retrieve a cognitive scaffold for the given task."""
    return get_reasoning(query)
```

---

## CrewAI

Inject reasoning as pre-task context for each crew member.

```python
import requests
from crewai import Agent, Task, Crew

injection = get_reasoning("Analyze root cause of production failures")

analyst = Agent(
    role="Production Analyst",
    goal="Identify the root cause of system failures",
    backstory=f"You are a production analyst.\n\n{injection}",
    llm=your_llm
)
```

For multi-agent crews, inject different scaffolds per agent. The root cause analyst gets a Causal scaffold. The timeline estimator gets a Temporal scaffold. The report writer gets an Abstraction scaffold.

---

## Claude Code / Agent SDK

### Via tool_use

```python
tools = [{
    "name": "get_reasoning_scaffold",
    "description": "Retrieve a cognitive scaffold for the current task",
    "input_schema": {
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Task description"},
            "mode": {"type": "string", "enum": ["single", "multi"], "default": "single"}
        },
        "required": ["query"]
    }
}]
```

When Claude decides to use this tool, make the POST request to the Ejentum API and return the scaffold as the tool result.

---

## Agentic IDEs (Cursor, Windsurf, Antigravity, Codex)

All major agentic IDEs support custom HTTP tools natively. No wrapper needed.

1. Add a custom tool definition pointing to the Ejentum POST endpoint
2. The IDE's agent calls the tool when it needs reasoning augmentation
3. The scaffold is injected into the agent's context automatically

This works identically across Cursor, Windsurf, Google Antigravity, and OpenAI Codex. Each IDE has its own tool configuration format, but the HTTP request is always the same: POST to `/logicv1/` with your query and API key.

---

## Make.com

1. **HTTP Module**: POST to the Ejentum endpoint with your query
2. **Text Aggregator**: Format the response into the injection template
3. **AI Module**: Paste the formatted text into the system message input

---

## Universal Pattern

Any framework. Any language. Three steps:

```
1. POST  https://ejentum-main-ab125c3.zuplo.app/logicv1/
   Body: {"query": "your task", "mode": "single"}
   Auth: Bearer YOUR_API_KEY

2. PARSE  response[0].single_ability  (or multi_ability)

3. INJECT  into system message before task prompt
```

---

## Advanced Patterns

### Task-Adaptive Injection

Different steps in a multi-step agent need different reasoning. Don't use one scaffold for the whole pipeline.

```python
tasks = [
    {"description": "Identify why production failed", "agent": analyst},
    {"description": "Estimate recovery timeline", "agent": planner},
    {"description": "Draft incident report", "agent": writer}
]

for task in tasks:
    injection = get_reasoning(task["description"])
    task["agent"].backstory = f"{task['agent'].base_backstory}\n\n{injection}"
```

The first task activates Causal reasoning. The second activates Temporal. The third activates Abstraction. One static scaffold would have forced all three agents into the same reasoning mode.

### Feedback Loop: Re-inject on Failure

If the agent's output fails validation, re-query with the failure description.

```python
result = agent.run(task)

if not validate(result):
    correction = get_reasoning(
        f"Agent failed: {validation_error}. Retry with corrective reasoning."
    )
    result = agent.run(task, system_override=correction)
```

This often triggers a Metacognitive scaffold (self-monitoring, contradiction detection) that was not selected on the first pass.

### Graceful Degradation

Always wrap the API call with a timeout and fallback. Your agent must function if the API is unreachable.

```python
def get_reasoning_safe(query: str) -> str:
    try:
        r = requests.post(EJENTUM_URL, json={"query": query, "mode": "single"},
                         headers={"Authorization": f"Bearer {EJENTUM_KEY}"}, timeout=2)
        r.raise_for_status()
        return f"[REASONING CONTEXT]\n{r.json()[0]['single_ability']}\n[END REASONING CONTEXT]"
    except Exception:
        return ""  # Agent continues with native reasoning
```

---

## Production Checklist

Before deploying:

- [ ] Wrap all API calls with timeout (2 seconds) and fallback
- [ ] Inject into system message, not user message
- [ ] Inject BEFORE task instructions, not after
- [ ] Test with representative queries from your actual pipeline (50+ tasks)
- [ ] Compare output quality with and without injection
- [ ] Re-inject per turn in multi-turn agents
- [ ] For compound reasoning: use `multi` mode, not multiple single calls
- [ ] Log responses to debug ability routing
- [ ] Graceful degradation: agent functions if API is unreachable
