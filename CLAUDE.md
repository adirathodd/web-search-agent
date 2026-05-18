# Web Search Agent

A learning project for understanding agentic AI loops in raw Python — no frameworks.

## Purpose

Build a web search agent from scratch using the Google Gemini API (`google-genai` SDK) and DuckDuckGo search. The goal is to understand how tool-calling and agentic loops work at the lowest level.

## Stack

- **LLM**: Google Gemini (free tier) via `google-genai` Python SDK
- **Search**: DuckDuckGo via `duckduckgo-search` Python package
- **Python**: 3.11+, sync (no async), no agent frameworks

## Project Structure

```
agent.py          # Phase 1: single-query agent loop
chat.py           # Phase 2: multi-turn REPL
tools.py          # Tool schema + DuckDuckGo executor
.env              # GEMINI_API_KEY (never commit this)
requirements.txt
```

## Key Concepts

- `tools.py` separates the **tool schema** (description for the model) from the **tool executor** (real Python code)
- `agent.py` contains a single explicit `while` loop — the entire agentic loop is visible in one place
- The `messages` list is the agent's memory — every tool call and result is appended to it

## Setup

```bash
pip install -r requirements.txt
cp .env.example .env  # add your GEMINI_API_KEY
python agent.py
```

## Development Notes

- Keep the loop explicit and readable — avoid abstractions that hide what's happening
- Print intermediate steps during development to observe loop behavior
- `.env` is gitignored — never commit API keys
