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

# RAG / embeddings
pip install voyageai      # for RAG/002_embeddings.ipynb (Claude side)
```

## Running notebooks

```powershell
jupyter notebook 001_requests.ipynb
```

There is no build, lint, or test pipeline — this is a notebook-first learning repo. Validate changes by running the affected notebook's cells top to bottom.

## API keys

All notebooks load from a `.env` file at the repo root (gitignored):

```
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=AIza...
VOYAGE_API_KEY=...
```

`Anthropic()` and `genai.Client()` auto-detect their respective keys from the environment — no need to pass them explicitly. Anthropic has no native embeddings API, so the RAG notebooks use Voyage AI (`voyage-3-large`) as the Claude-side embedding provider, while Gemini uses its own native embedding endpoint.

## Repo layout

- Root: baseline request/prompting/eval notebook pairs (see below).
- `Tool_use_with_Claude/`: tool-use / function-calling notebooks, Claude and Gemini side by side.
- `RAG/`: chunking and embeddings notebooks, plus `report.md` — a synthetic long-form document used only as sample text for chunking/embedding demos (not real content).
- `dataset.json`, `output.json`, `output.html`: generated artifacts from the eval notebooks. Regenerating overwrites prior content — avoid noisy churn from these in unrelated edits.

Every topic follows the same shape: a Claude notebook and a Gemini notebook (or notebook pair) implementing the equivalent workflow, so the two SDKs can be compared directly. When editing one side, check whether the equivalent change belongs on the other side too.

## Core request pattern

`001_requests.ipynb` (anthropic, `claude-sonnet-4-6`) and `002_gemini_requests.ipynb` (google-genai, `gemini-2.5-flash`) each build a `messages` list via `add_user_message` / `add_assistant_message` helpers, sent through a `chat()` function — a multi-turn conversation loop. `chat()` supports `system`, `temperature` (Anthropic only), and `stop_sequences` for both SDKs, but the shape differs: Anthropic passes these as flat kwargs to `messages.create`; Gemini wraps `system_instruction` / `stop_sequences` into a `types.GenerateContentConfig` passed as `config`. This helper pattern (message-list + `chat()`) is reused as the foundation across the prompting, eval, and tool-use notebooks.

## Prompting & eval notebooks

| Notebook | SDK | Notes |
|---|---|---|
| `001_prompting.ipynb` | anthropic, `claude-haiku-4-5` | `PromptEvaluator` class: dataset generation, concurrent grading, HTML report |
| `002_gemini_prompting.ipynb` | google-genai, `gemini-3.1-flash-lite` | Gemini port of `001_prompting.ipynb`; task: 1-day athlete meal-plan prompt |
| `003_gemini_prompting.ipynb` | google-genai, `gemini-3.1-flash-lite` | Same scaffold as 002; task: extract topics into a JSON array of strings |
| `001_prompt_evals.ipynb` | anthropic + google-genai | Builds and calls `generate_dataset()` only |
| `002_gemini_prompt_evals.ipynb` | google-genai | Full eval loop (see below) |
| `001_prompt_evals_fns.ipynb` | anthropic | AWS Python/JSON/Regex dataset; grades via LLM judge **and** structural validator functions (`validate_json`, `validate_python`, regex) |
| `002_gemini_prompt_evals_fns.ipynb` | google-genai | Gemini counterpart of `001_prompt_evals_fns.ipynb` |
| `001_prompt_evals_grader.ipynb` | anthropic | Same AWS dataset/pipeline, but scores via `grade_by_model` (LLM judge) only — no structural validators |

`001_prompt_evals.ipynb` wraps the Gemini side in a `GeminiChat` class (same method names as the Anthropic module-level helpers, but as instance methods) so both providers can be driven side by side. `002_gemini_prompt_evals.ipynb` implements the full eval loop: `generate_dataset()` → persist to `dataset.json` → reload it → `run_prompt(test_case)` (solves the task) → `grade_by_model(test_case, output)` (LLM-as-judge, same `stop_sequences=["```"]` JSON-forcing trick as `generate_dataset`) → `run_test_case()` combines both → `run_eval(dataset)` maps it over the whole dataset. `dataset.json` is a generated artifact, not hand-authored — regenerating it overwrites prior tasks. If porting the eval loop back into `001_prompt_evals.ipynb`, follow this same generate → persist → reload → run → grade shape.

In `_fns`/`_grader` notebooks, "fns" refers to the structural validator functions, not tool/function-calling — don't confuse these with the `Tool_use_with_Claude/` notebooks.

## Tool use (`Tool_use_with_Claude/`)

Ordering is by leading number (`001` → single tool, `002`/`003` → multiple tools, `003` → streaming, `004` → multi-tool dispatcher example); trailing suffixes like `_007`/`_009` are leftover autosave/version numbers with no meaning — ignore them.

- `001_tools.ipynb` / `001_tools_007.ipynb` (Claude): single tool definition and schema; `_007` is the expanded version with an added `get_current_datetime` tool and typed `ToolParam`/`Message` helpers.
- `002_gemini_tools.ipynb` (Gemini): single-tool equivalent using `types.Tool(function_declarations=...)` with manual `function_call`/`function_response` handling.
- `003_gemini_multiple_tools.ipynb` / `004_geminis_tools_009.ipynb` (Gemini): multiple tools with a `run_tool` dispatcher and a `run_conversation` loop.
- `003_tool_streaming.ipynb` (Claude, `claude-sonnet-4-5`): streaming tool use via `chat_stream`.
- `003_gemini_tool_streaming.ipynb` (Gemini): streaming equivalent, using `FunctionDeclaration` and accumulating streamed function-call parts.

## RAG (`RAG/`)

`001_chunking.ipynb` implements three chunking strategies — fixed character count with overlap, sentence-based with overlap, and markdown `## `-section splitting — applied to `report.md` (synthetic sample text, not real content). `002_embeddings.ipynb` (Voyage AI, `voyage-3-large`) and `002_gemini_embeddings.ipynb` (Gemini, `gemini-embedding-*`, `task_type="RETRIEVAL_QUERY"`) both reuse the section-chunking function and embed the resulting chunks.

## Purpose

Learning/exploration notebooks comparing the Anthropic and Google Gemini Python SDKs side by side, across requests, prompting/eval loops, tool use, and RAG chunking/embeddings.
