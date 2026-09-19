# How to Build Your First MCP Server for Claude

*A beginner's guide to connecting Claude to your own tools and data*

## What is MCP and why it matters

The Model Context Protocol (MCP) is an open standard for connecting AI models to external tools and data sources — things like your notes, a database, or a custom API — without writing one-off integration code for every single connection.

Before MCP, giving Claude access to, say, your file system or a search tool meant custom glue code specific to that one integration. MCP standardizes this: build one MCP server, and any MCP-compatible client (Claude Desktop, Claude Code, and others) can use it the same way.

By the end of this guide, you'll have a working local MCP server exposing one simple tool, connected to Claude Desktop and ready to test. Real first-timers report this taking about 30 minutes once you know the steps — this guide exists so you skip the trial and error.

## Prerequisites

Before you start, make sure you have:

- **Python 3.11 or higher** installed (`python --version` to check)
- **Claude Desktop** installed and signed in
- A terminal and a text editor
- No prior MCP experience needed — this guide assumes zero background beyond basic Python

We'll use **FastMCP**, a Python library that handles the protocol details for you so you can focus on what your tool actually does.

## Step 1: Set up your project

Create a new folder for your server and install FastMCP:

```bash
mkdir my-first-mcp-server
cd my-first-mcp-server
pip install fastmcp
```

That's the entire setup step — no build tools, no config files yet.

## Step 2: Define your first tool

Create a file called `server.py`. We'll build a simple tool that searches a small, hardcoded list of notes — enough to prove the connection works end-to-end without needing a real database.

```python
from fastmcp import FastMCP
import logging

# Log to stderr, not stdout — see "Common Pitfalls" for why this matters
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
)
logger = logging.getLogger(__name__)

mcp = FastMCP("NoteSearch")

NOTES = [
    {"id": "note_1", "title": "MCP Ideas", "content": "Build a demo MCP server"},
    {"id": "note_2", "title": "Project Tasks", "content": "Write blog posts, review PRs"},
]

@mcp.tool
def search_notes(query: str) -> list[dict]:
    """Search notes by keyword and return matching results."""
    logger.info(f"Searching notes for: {query}")
    return [n for n in NOTES if query.lower() in n["title"].lower() or query.lower() in n["content"].lower()]

if __name__ == "__main__":
    mcp.run()
```

A few things worth noticing here: the `@mcp.tool` decorator is doing most of the work — it reads your function's type hints and docstring to automatically build the tool description Claude sees. Write a clear docstring; it's not just a comment, it's documentation Claude actually reads to decide when to use your tool.

## Step 3: Connect it to Claude Desktop

Claude Desktop reads a config file to know which MCP servers to launch. Find yours here:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux:** `~/.config/Claude/claude_desktop_config.json`

If the file doesn't exist yet, create it with `{"mcpServers": {}}` as a starting point. Then add your server:

```json
{
  "mcpServers": {
    "note-search": {
      "command": "python",
      "args": ["/absolute/path/to/your/server.py"]
    }
  }
}
```

**Use the full absolute path** to `server.py` — a relative path is one of the most common reasons this silently fails.

## Step 4: Test it end-to-end

**Fully quit and reopen Claude Desktop** (not just closing the window — MCP servers are only loaded on startup).

Once it's back open, look for a small tool/hammer icon in the chat interface — that's your signal MCP servers are connected. Then just ask Claude something naturally:

> "Can you search my notes for anything about MCP?"

If it's working, Claude will call your `search_notes` tool and return the matching note. That's the whole loop: your Python function, exposed through MCP, callable by Claude like any other tool.

## Common pitfalls

A few things that trip up almost everyone on their first server:

1. **Using `print()` for debugging.** MCP over stdio uses standard output to send protocol messages — a stray `print()` statement corrupts that stream and breaks the connection. Use Python's `logging` module configured to write to stderr instead (as shown in Step 2).
2. **Forgetting the full restart.** Editing the config file while Claude Desktop is still running does nothing until you quit and relaunch it completely.
3. **Relative paths in the config file.** Always use the absolute path to your script — Claude Desktop doesn't run from your project folder, so relative paths resolve incorrectly.
4. **Invalid JSON in the config file.** A single trailing comma or missing bracket will silently prevent *every* server in the file from loading, not just the broken entry. Validate your JSON before restarting.
5. **Thin docstrings.** Since FastMCP builds the tool's description from your docstring, a vague one ("does stuff") makes it harder for Claude to know when to actually use your tool.

## Next steps

You've built a server with one tool — MCP also supports **resources** (read-only data Claude can reference) and **prompts** (reusable templates), both worth exploring once this basic loop feels comfortable. From here, a natural next step is connecting a tool to something real — an actual database, a file system, or an API — instead of the hardcoded list used in this guide.
