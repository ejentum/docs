# Integrations

Ejentum is a single POST endpoint returning JSON. Any system that can make an HTTP request and inject text into a prompt works with the Logic API. No SDK required. No dependencies.

**Choose your path:** [No code (n8n)](#n8n) · [Python (LangChain)](#langchain--langgraph) · [Multi-agent (CrewAI)](#crewai) · [Claude (Agent SDK)](#claude-code--agent-sdk) · [IDE (Cursor/Windsurf)](#agentic-ides-cursor-windsurf-antigravity-codex) · [Make.com](#makecom) · [Any framework](#universal-pattern)

For endpoint details, see the [API Reference](/docs/api_reference). For real injection payloads, see [Examples](/docs/examples).

## The Injection Principle

Where you inject matters as much as what you inject.

**1. Inject BEFORE the task, not after.**
The injection must be the first structured content the model processes. LLMs attend most strongly to content at the beginning of context.

**2. Inject into the SYSTEM message, not the user message.**
The system message sets the model's operational mode. Injecting into the user message treats the injection as data to reason about, not as a constraint to follow.

**3. Keep the injection SEPARATE from your instructions.**
The `[REASONING CONTEXT]...[END REASONING CONTEXT]` delimiters create a distinct attention block. Do not merge the injection into natural language instructions.

**4. Re-inject per turn in multi-turn agents.**
Injections degrade over long contexts. Call the API for each new task step and inject fresh. Injections act as persistent attention anchors, but they lose effectiveness as task-specific tokens accumulate over extended chains. Re-injection maintains the effect.

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
   - Body: `{"query": "{task_description}", "mode": "reasoning"}`
4. The agent receives the injection in the tool response and uses it to guide its reasoning

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

def get_injection(task: str, mode: str = "reasoning") -> str:
    try:
        r = requests.post(
            EJENTUM_URL,
            headers={"Authorization": f"Bearer {EJENTUM_KEY}", "Content-Type": "application/json"},
            json={"query": task, "mode": mode},
            timeout=2
        )
        r.raise_for_status()
        injection = r.json()[0][mode]
        return f"[REASONING CONTEXT]\n{injection}\n[END REASONING CONTEXT]"
    except Exception:
        return ""

prompt = ChatPromptTemplate.from_messages([
    ("system", "{reasoning}\n\nYou are a senior analyst."),
    ("human", "{task}")
])

chain = (
    RunnablePassthrough.assign(reasoning=lambda x: get_injection(x["task"]))
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
    """Retrieve a cognitive ability for the given task."""
    return get_injection(query)
```

---

## CrewAI

Inject reasoning as pre-task context for each crew member.

```python
import requests
from crewai import Agent, Task, Crew

injection = get_injection("Analyze root cause of production failures")

analyst = Agent(
    role="Production Analyst",
    goal="Identify the root cause of system failures",
    backstory=f"You are a production analyst.\n\n{injection}",
    llm=your_llm
)
```

For multi-agent crews, inject different abilities per agent. The root cause analyst gets a Causal ability. The timeline estimator gets a Temporal ability. The report writer gets an Abstraction ability.

---

## Claude Code / Agent SDK

### Via tool_use

```python
tools = [{
    "name": "get_ejentum_injection",
    "description": "Retrieve a cognitive ability for the current task",
    "input_schema": {
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Task description"},
            "mode": {"type": "string", "enum": ["reasoning", "reasoning-multi", "anti-deception", "code", "code-multi", "memory", "memory-multi"], "default": "reasoning"}
        },
        "required": ["query"]
    }
}]
```

When Claude decides to use this tool, make the POST request to the Ejentum API and return the injection as the tool result.

---

## Agentic IDEs (Cursor, Windsurf, Antigravity, Codex)

All major agentic IDEs support custom HTTP tools natively. No wrapper needed.

1. Add a custom tool definition pointing to the Ejentum POST endpoint
2. The IDE's agent calls the tool when it needs reasoning augmentation
3. The injection is placed into the agent's context automatically

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
   Body: {"query": "your task", "mode": "reasoning"}
   Auth: Bearer YOUR_API_KEY

2. PARSE  response[0][mode]  (key matches mode name)

3. INJECT  into system message before task prompt
```

---

## Advanced Patterns

### Task-Adaptive Injection

Different steps in a multi-step agent need different reasoning. Don't use one injection for the whole pipeline.

```python
tasks = [
    {"description": "Identify why production failed", "agent": analyst},
    {"description": "Estimate recovery timeline", "agent": planner},
    {"description": "Draft incident report", "agent": writer}
]

for task in tasks:
    injection = get_injection(task["description"])
    task["agent"].backstory = f"{task['agent'].base_backstory}\n\n{injection}"
```

The first task activates Causal reasoning. The second activates Temporal. The third activates Abstraction. One static injection would have forced all three agents into the same reasoning mode.

### Feedback Loop: Re-inject on Failure

If the agent's output fails validation, re-query with the failure description.

```python
result = agent.run(task)

if not validate(result):
    correction = get_injection(
        f"Agent failed: {validation_error}. Retry with corrective reasoning."
    )
    result = agent.run(task, system_override=correction)
```

This often triggers a Metacognitive ability (self-monitoring, contradiction detection) that was not selected on the first pass.

### Graceful Degradation

Always wrap the API call with a timeout and fallback. Your agent must function if the API is unreachable.

```python
def get_injection_safe(query: str, mode: str = "reasoning") -> str:
    try:
        r = requests.post(EJENTUM_URL, json={"query": query, "mode": mode},
                         headers={"Authorization": f"Bearer {EJENTUM_KEY}"}, timeout=2)
        r.raise_for_status()
        payload = r.json()[0].get(mode, "")
        return f"[REASONING CONTEXT]\n{payload}\n[END REASONING CONTEXT]" if payload else ""
    except Exception:
        return ""  # Agent continues with native capability
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

---

**See also:** [Use Cases](/use-cases) for industry-specific integration patterns. [Builder's Playbook](/docs/builders_playbook) for real-world workflow examples.
