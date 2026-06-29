# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment

Python virtualenv is at `.venv/`. Activate it before running anything:

```powershell
.venv\Scripts\activate.ps1
```

Install dependencies:

```powershell
pip install anthropic python-dotenv
```

## Running the notebook

```powershell
jupyter notebook 001_requests.ipynb
```

## API key

The project uses `python-dotenv` to load a `.env` file. Create one at the repo root:

```
ANTHROPIC_API_KEY=sk-ant-...
```

Then in the notebook:

```python
from dotenv import load_dotenv
import os
load_dotenv()
client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
```

## Purpose

Learning/exploration notebooks for the Anthropic Python SDK. `001_requests.ipynb` covers basic API requests using `anthropic` client.
