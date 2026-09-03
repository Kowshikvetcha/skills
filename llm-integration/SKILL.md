---
name: llm-integration
description: >
  Generates a ready-to-use, zero-edit Python LLM integration for any project. Use this skill
  whenever the user wants to add AI/LLM capabilities to their Python project, connect to an AI
  API, set up a chat or completion client, or asks about integrating OpenAI, Anthropic, Google
  Gemini, Mistral, Groq, Ollama, OpenRouter, or HuggingFace. Also trigger for phrases like
  "add AI to my project", "LLM wrapper", "AI client setup", "multi-provider AI", or
  "model-agnostic LLM". API key goes in .env, all LLM config goes in llm_config.yaml —
  the user never edits the Python code. Only required inputs: provider name and AI_API_KEY.
---

# LLM Integration Skill

Generates **four ready-to-use files** that drop into any Python project:

| File | Purpose | User edits? |
|---|---|---|
| `llm_client.py` | Unified LLM client — all providers | ❌ Never |
| `llm_config.yaml` | Provider, model, temperature, etc. | ✅ Yes |
| `.env` | API key only | ✅ Just paste key |
| `requirements.txt` | pip dependencies | ❌ Just run it |

---

## Required inputs (ask if not provided)

| Field | Description |
|---|---|
| `provider` | One of: `openai`, `anthropic`, `gemini`, `mistral`, `groq`, `ollama`, `openrouter`, `huggingface` |
| `AI_API_KEY` | The user's API key (written into `.env`; use `"ollama"` for Ollama) |

## Optional inputs (use defaults silently — never ask)

| Field | Default |
|---|---|
| `model` | See provider defaults table |
| `temperature` | `0.7` |
| `max_tokens` | `1024` |
| `system_prompt` | `"You are a helpful assistant."` |

## Provider defaults table

| Provider | Default model | Notes |
|---|---|---|
| `openai` | `gpt-4o-mini` | OpenAI-compatible REST |
| `anthropic` | `claude-3-5-haiku-20241022` | Native SDK; system prompt is top-level param |
| `gemini` | `gemini-1.5-flash` | `google-generativeai` SDK; role is `model` not `assistant` |
| `mistral` | `mistral-small-latest` | Native `mistralai` SDK |
| `groq` | `llama-3.1-8b-instant` | OpenAI-compatible |
| `ollama` | `llama3.2` | OpenAI-compatible, local server, no real key needed |
| `openrouter` | `mistralai/mistral-7b-instruct` | OpenAI-compatible; model names include provider prefix |
| `huggingface` | `microsoft/DialoGPT-medium` | `huggingface_hub` SDK |

---

## Generation workflow

### Step 1 — Collect inputs

Ask for `provider` and `AI_API_KEY` if not provided. Accept optional fields if given;
otherwise use defaults from the table above. Never ask about model, temperature, etc.

### Step 2 — Read references

Read both reference files before writing any code:
- `references/providers.md` — per-provider SDK details and auth patterns
- `references/unified_interface.md` — exact file structure and class contract

### Step 3 — Generate all four files

#### `llm_config.yaml`
Pre-filled with the chosen provider and all defaults. User edits this file to change settings.
```yaml
# LLM Configuration
# Edit this file to change provider, model, or generation settings.
# Your API key lives in .env — do not add it here.

provider: <chosen-provider>

model: <default-model>          # optional: delete this line to use the built-in default
temperature: 0.7                # 0.0 (deterministic) to 1.0 (creative)
max_tokens: 1024                # maximum tokens in the response
system_prompt: "You are a helpful assistant."
```

#### `.env`
Pre-filled with the user's actual key:
```
# LLM API Key — never commit this file to git!
AI_API_KEY=<user-provided-key>
```

#### `llm_client.py`
Fully self-contained, zero-edit Python file. It:
1. Loads `.env` via `python-dotenv` → reads `AI_API_KEY`
2. Loads `llm_config.yaml` via `PyYAML` → reads provider + all config
3. Validates both are present and non-empty, with helpful error messages
4. Builds the correct provider backend internally based on `provider` field
5. Exposes a clean `LLMClient` class — **constructor takes zero arguments**

User-facing usage is simply:
```python
from llm_client import LLMClient
client = LLMClient()          # reads .env and llm_config.yaml automatically
client.chat("Hello!")
```

See `references/unified_interface.md` for the full class contract and file structure.

#### `requirements.txt`
Always includes `python-dotenv` and `pyyaml`. Add provider-specific packages from
`references/providers.md`.

### Step 4 — Output files

Save all four to `/mnt/user-data/outputs/`:
- `llm_client.py`
- `llm_config.yaml`
- `.env`
- `requirements.txt`

Present all four files to the user.

---

## After delivering the files

Tell the user:
1. **Install:** `pip install -r requirements.txt`
2. **API key:** Already in `.env` — add `.env` to `.gitignore` and never commit it.
3. **Change model/settings:** Edit `llm_config.yaml` — no Python changes needed.
4. **Use in project:**
   ```python
   from llm_client import LLMClient
   client = LLMClient()
   print(client.chat("Hello!"))
   ```
5. **Run built-in demo:** `python llm_client.py`
6. Any provider-specific setup notes (e.g. Ollama needs `ollama serve` running locally;
   HuggingFace gated models require Pro account; OpenRouter requires model name prefix).
