# Web Search Agent

A web search agent built in raw Python — no frameworks — to understand how agentic AI loops work from first principles.

## What This Is

This project uses the Google Gemini API and DuckDuckGo to build an agent that can answer questions by searching the web. The goal is not a polished product, but a transparent implementation where every step of the agentic loop is visible and understandable.

## Stack

| Component | Choice | Why |
|-----------|--------|-----|
| LLM | Google Gemini (free tier) | Generous free tier, native tool-calling support |
| Search | DuckDuckGo (`duckduckgo-search`) | Free, no API key, forces you to handle tool mechanics manually |
| SDK | `google-genai` | Native Gemini SDK, exposes tool-calling format directly |

## Architecture

```
web-search-agent/
├── agent.py          # Phase 1: single-query agent loop
├── chat.py           # Phase 2: multi-turn REPL
├── tools.py          # Tool schema + DuckDuckGo executor
├── .env              # GEMINI_API_KEY (gitignored)
└── requirements.txt
```

**`tools.py`** has two distinct responsibilities — deliberately kept together so the contrast is clear:
- The **tool schema**: a dict that describes the tool to the model (name, description, parameters)
- The **tool executor**: the actual Python function that calls DuckDuckGo and returns results

**`agent.py`** contains a single `run_agent(query: str) -> str` function with an explicit `while` loop. No classes, no abstractions — the full loop is readable top to bottom.

**`chat.py`** extends the same loop logic with a persistent message history across turns.

## How the Agentic Loop Works

This is the core concept the project is built to teach:

```
1. Pass a query string
         ↓
2. Build messages list: [{"role": "user", "content": query}]
         ↓
3. Send to Gemini with tool definitions attached
         ↓
4. Gemini responds with one of two outcomes:
   a) TOOL_CALL → model wants to search something
   b) TEXT      → model has a final answer, exit loop
         ↓ (if TOOL_CALL)
5. Extract tool name + arguments from the response
         ↓
6. Execute the tool (call DuckDuckGo, get results string)
         ↓
7. Append model's tool call AND your tool result to messages list
         ↓
8. Go back to step 3 — loop again with updated history
```

The `messages` list is the agent's entire memory. Every loop iteration appends to it — that's how the model knows what it already searched and what it found. For multi-turn chat (Phase 2), this list persists across queries instead of resetting.

## Setup

```bash
git clone https://github.com/adirathodd/web-search-agent
cd web-search-agent
pip install -r requirements.txt

# Add your Gemini API key
echo "GEMINI_API_KEY=your_key_here" > .env

# Phase 1: single query
python agent.py

# Phase 2: multi-turn chat
python chat.py
```

## Key Learning Concepts

- **Tool schema vs tool executor** — the model never runs your code; it returns a structured request, and you decide whether and how to execute it
- **Messages list as memory** — the full conversation history (including tool calls and results) is what gives the model context on each loop iteration
- **Loop termination** — the agent stops when the model returns plain text instead of a tool call; your loop checks for this condition
- **Multi-turn extension** — the only difference between single-query and chat is whether you reset the messages list between queries
