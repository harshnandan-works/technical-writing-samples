# LangChain Without the Headache: A Simplified First Agent

*LangChain's own docs cover a lot of ground — this guide covers just enough to get a working agent running, nothing more.*

## Why this guide exists

LangChain is one of the most widely used frameworks for building with LLMs, but its documentation has a well-earned reputation among developers for being sprawling — the library moves fast, and older tutorials floating around the internet often show syntax that's already outdated. This guide sticks to the current, minimal path: one working agent, in about 10 minutes, with nothing extra.

## Prerequisites

- Python 3.9 or higher
- An Anthropic API key (used as the model provider in this guide)
- Basic comfort reading Python — no prior LangChain experience needed

## Step 1: Install and set your API key

```bash
pip install -qU "langchain[anthropic]"
```

Then set your API key as an environment variable:

```bash
export ANTHROPIC_API_KEY="your-key-here"
```

(On Windows, use `set ANTHROPIC_API_KEY=your-key-here` in Command Prompt, or `$env:ANTHROPIC_API_KEY="your-key-here"` in PowerShell.)

## Step 2: Build your first agent

LangChain's current recommended entry point is `create_agent` — it wraps the model, a set of tools, and the agent loop into one call, instead of assembling those pieces manually.

```python
from langchain.agents import create_agent

def get_weather(city: str) -> str:
    """Get the weather for a given city."""
    # In a real app, this would call a weather API.
    # Hardcoded here so the example runs with zero extra setup.
    return f"It's sunny and 72°F in {city}."

agent = create_agent(
    model="claude-sonnet-4-5",
    tools=[get_weather],
    system_prompt="You are a helpful weather assistant.",
)

response = agent.invoke(
    {"messages": [{"role": "user", "content": "What's the weather like in Tokyo?"}]}
)

print(response["messages"][-1].content)
```

Run this file, and the agent will call `get_weather` on its own and respond with the result — that's the entire agent loop: model, tool, and reasoning about when to use it, in about 15 lines.

## Step 3: The other core pattern — a simple chain

Not everything needs to be an agent. For a single-purpose task (like translating text or summarizing something), LangChain's "LCEL" pattern — piping components together with `|` — is the simpler building block worth knowing:

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

model = ChatAnthropic(model="claude-sonnet-4-5")
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that explains concepts clearly and concisely."),
    ("user", "Explain {topic} in simple terms."),
])
parser = StrOutputParser()

chain = prompt | model | parser
result = chain.invoke({"topic": "how MCP servers work"})
print(result)
```

Read this left to right: the prompt gets filled in, passed to the model, and the model's response gets parsed into a plain string. Same idea as Step 2, just without the tool-calling loop.

## Common pitfalls

1. **Mismatched provider installs.** `pip install langchain` alone doesn't include a specific model provider — you need the extra, like `langchain[anthropic]` or the separate `langchain-anthropic` package, matching whichever model you're actually calling.
2. **Copying code from older tutorials.** LangChain has gone through several API redesigns. If a blog post's syntax looks meaningfully different from what's here, check its publish date before assuming it's still current.
3. **Wrong or missing environment variable name.** Each provider expects its own variable (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc.) — a silent auth failure is often just the wrong variable name.
4. **Assuming you need LangGraph on day one.** LangGraph (a related, more advanced library for complex multi-step agent workflows) is often mentioned alongside LangChain, but it's not required to get a basic agent like this one running.

## Next steps

Once this pattern feels comfortable, the natural next additions are giving your agent more than one tool, adding memory so it remembers earlier turns in a conversation, and — only once you actually need multi-step branching logic — looking at LangGraph.
