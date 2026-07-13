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
jupyter notebook 001_prompt_evals.ipynb
jupyter notebook 002_gemini_prompt_evals.ipynb
```

## API keys

All notebooks load from a `.env` file at the repo root:

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
| `001_prompt_evals.ipynb` | `anthropic` + `google-genai` | `claude-haiku-4-5` / `gemini-2.5-flash` |
| `002_gemini_prompt_evals.ipynb` | `google-genai` | `gemini-2.5-flash` |

`001_requests.ipynb` and `002_gemini_requests.ipynb` each follow the same pattern: a `messages` list built with `add_user_message` / `add_assistant_message` helpers, then sent through a `chat()` function. This mirrors a multi-turn conversation loop. `chat()` supports `system`, `temperature` (Anthropic only), and `stop_sequences` for both SDKs — note the shape differs: Anthropic passes these as flat kwargs to `messages.create`, Gemini wraps `system_instruction`/`stop_sequences` into a `types.GenerateContentConfig` passed as `config`.

`001_prompt_evals.ipynb` wraps the Gemini side in a `GeminiChat` class (same method names as the Anthropic module-level helpers, but as instance methods) so both providers can be driven side by side. `002_gemini_prompt_evals.ipynb` is its pure-Gemini counterpart — same `generate_dataset()` prompt, but using `002_gemini_requests.ipynb`'s module-level helper style instead of the class wrapper. Both use this setup to generate evaluation datasets (via prompted JSON generation) for grading model output on Python/JSON/Regex generation tasks for AWS-related work.

`001_prompt_evals.ipynb` only builds and calls `generate_dataset()`. `002_gemini_prompt_evals.ipynb` goes further and implements the full eval loop: `generate_dataset()` → persist to `dataset.json` at the repo root → reload it → `run_prompt(test_case)` (solves the task) → `grade_by_model(test_case, output)` (LLM-as-judge, same `stop_sequences=["```"]` JSON-forcing trick as `generate_dataset`) → `run_test_case()` combines both → `run_eval(dataset)` maps it over the whole dataset. `dataset.json` is a generated artifact, not hand-authored — regenerating it overwrites prior tasks. If porting the eval loop back into `001_prompt_evals.ipynb`, follow this same generate → persist → reload → run → grade shape.

## Purpose

Learning/exploration notebooks comparing the Anthropic and Google Gemini Python SDKs side by side.
