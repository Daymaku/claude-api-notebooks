# AGENTS.md

Agent instructions for this repository.

## Scope

- This is a notebook-first learning repo for Anthropic and Gemini SDK usage.
- There is no application build pipeline or automated test suite in this workspace.
- Prefer minimal, targeted edits and preserve notebook flow.

## Quick Start

1. Activate environment:
   - `.venv\Scripts\activate.ps1`
2. Install provider dependencies as needed:
   - `pip install anthropic python-dotenv`
   - `pip install google-genai python-dotenv`
3. Ensure API keys exist in root `.env`:
   - `ANTHROPIC_API_KEY=...`
   - `GEMINI_API_KEY=...`

Reference: [CLAUDE.md](CLAUDE.md)

## Canonical File Map

- Anthropic request baseline: [001_requests.ipynb](001_requests.ipynb)
- Gemini request baseline: [002_gemini_requests.ipynb](002_gemini_requests.ipynb)
- Anthropic prompt-eval workflow: [001_prompt_evals.ipynb](001_prompt_evals.ipynb)
- Gemini prompt-eval workflow: [002_gemini_prompt_evals.ipynb](002_gemini_prompt_evals.ipynb)
- Extended Gemini prompting/eval flow: [003_gemini_prompting.ipynb](003_gemini_prompting.ipynb)

## Working Conventions

- Reuse the existing message-helper plus `chat()` pattern in request notebooks.
- Keep provider differences explicit:
  - Anthropic uses flat kwargs to `messages.create`.
  - Gemini uses `GenerateContentConfig` for settings like `system_instruction` and `stop_sequences`.
- Keep JSON-forcing behavior intact where used (assistant prefill plus `stop_sequences=["```"]`) unless intentionally redesigning parser flow.

## Generated Artifacts

- [dataset.json](dataset.json), [output.json](output.json), and [output.html](output.html) are generated artifacts.
- Regeneration overwrites previous content. Only regenerate when explicitly needed.
- Avoid noisy churn in generated outputs during unrelated edits.

## Notebook Editing Rules

- Prefer editing only the minimum required cells.
- Keep cell order and execution flow stable unless the task requires a structural refactor.
- If changing evaluation concurrency settings, keep defaults conservative to avoid API rate-limit failures.

## Secrets and Safety

- Never hardcode or commit API keys.
- Use root `.env` and provider auto-discovery.

## Additional Context

- Main project guidance is documented in [CLAUDE.md](CLAUDE.md). Link to it instead of duplicating long guidance.
