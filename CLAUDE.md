# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment

Python virtualenv is at `.venv/`. Activate it before running anything:

```powershell
.venv\Scripts\activate.ps1
```

Install dependencies per notebook:

```powershell
# Anthropic
pip install anthropic python-dotenv

# Gemini
pip install google-genai python-dotenv
```

## Running notebooks

```powershell
jupyter notebook 001_requests.ipynb
jupyter notebook 002_gemini_requests.ipynb
```

## API keys

Both notebooks load from a `.env` file at the repo root:

```
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=AIza...
```

Both `Anthropic()` and `genai.Client()` auto-detect their respective keys from the environment — no need to pass them explicitly.

## Notebooks

| Notebook | SDK | Default model |
|---|---|---|
| `001_requests.ipynb` | `anthropic` | `claude-sonnet-4-6` |
| `002_gemini_requests.ipynb` | `google-genai` | `gemini-2.5-flash` |

Both follow the same pattern: a `messages` list built with `add_user_message` / `add_assistant_message` helpers, then sent through a `chat()` function. This mirrors a multi-turn conversation loop.

## Purpose

Learning/exploration notebooks comparing the Anthropic and Google Gemini Python SDKs side by side.
