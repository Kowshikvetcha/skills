# Unified Interface Reference

Defines the exact structure `llm_client.py` must follow for every provider.

---

## File structure (in order)

```
1. Module docstring   — what this file is, how to configure it
2. Imports            — stdlib + provider SDK + dotenv + yaml
3. _load_config()     — private function: loads .env and llm_config.yaml
4. _PROVIDER_DEFAULTS — dict mapping provider name → default model
5. LLMClient class    — see full spec below
6. __main__ block     — demo: instantiates LLMClient() and runs tests
```

---

## _load_config() function

This function is always present and always looks like this:

```python
import os
import yaml
from pathlib import Path
from dotenv import load_dotenv

def _load_config() -> dict:
    """
    Load API key from .env and LLM settings from llm_config.yaml.
    Both files must be in the same directory as this script or the working directory.
    """
    # Load .env — sets AI_API_KEY in os.environ
    env_path = Path(__file__).parent / ".env"
    load_dotenv(dotenv_path=env_path)

    api_key = os.getenv("AI_API_KEY", "").strip()
    if not api_key:
        raise ValueError(
            "AI_API_KEY not found. Make sure .env exists and contains:\n"
            "  AI_API_KEY=your-key-here"
        )

    # Load llm_config.yaml
    config_path = Path(__file__).parent / "llm_config.yaml"
    if not config_path.exists():
        raise FileNotFoundError(
            f"llm_config.yaml not found at {config_path}.\n"
            "Create it with at least: provider: <provider-name>"
        )

    with open(config_path) as f:
        config = yaml.safe_load(f) or {}

    provider = config.get("provider", "").strip().lower()
    if not provider:
        raise ValueError("llm_config.yaml must contain a 'provider' field.")

    return {
        "api_key": api_key,
        "provider": provider,
        "model": config.get("model"),              # None = use built-in default
        "temperature": float(config.get("temperature", 0.7)),
        "max_tokens": int(config.get("max_tokens", 1024)),
        "system_prompt": config.get("system_prompt", "You are a helpful assistant."),
    }
```

---

## _PROVIDER_DEFAULTS dict

Always present, always maps provider string to default model name:

```python
_PROVIDER_DEFAULTS = {
    "openai":       "gpt-4o-mini",
    "anthropic":    "claude-3-5-haiku-20241022",
    "gemini":       "gemini-1.5-flash",
    "mistral":      "mistral-small-latest",
    "groq":         "llama-3.1-8b-instant",
    "ollama":       "llama3.2",
    "openrouter":   "mistralai/mistral-7b-instruct",
    "huggingface":  "microsoft/DialoGPT-medium",
}
```

---

## LLMClient class (full spec)

```python
class LLMClient:
    """
    Model-agnostic LLM client. Configuration is loaded automatically from:
      - .env          → AI_API_KEY
      - llm_config.yaml → provider, model, temperature, max_tokens, system_prompt

    Usage:
        client = LLMClient()                      # zero arguments
        response = client.chat("Hello!")
        response = client.chat_with_history([...])
        for chunk in client.stream("Hello!"):
            print(chunk, end="", flush=True)
    """

    def __init__(self):
        config = _load_config()

        self.provider      = config["provider"]
        self.model         = config["model"] or _PROVIDER_DEFAULTS.get(self.provider)
        self.temperature   = config["temperature"]
        self.max_tokens    = config["max_tokens"]
        self.system_prompt = config["system_prompt"]
        self._api_key      = config["api_key"]

        if self.model is None:
            raise ValueError(
                f"Unknown provider '{self.provider}'. "
                f"Supported: {list(_PROVIDER_DEFAULTS.keys())}"
            )

        # Build the provider-specific backend client
        self._backend = self._init_backend()

    def _init_backend(self):
        """Initialize and return the provider SDK client object."""
        # provider-specific initialization here
        # return the raw SDK client (OpenAI(), Anthropic(), etc.)
        ...

    def _build_messages(self, message: str) -> list[dict]:
        """Wrap a single user string into the messages list format."""
        return [{"role": "user", "content": message}]

    def chat(self, message: str) -> str:
        """
        Send a single message and return the assistant's reply as a string.

        Args:
            message (str): The user's message.

        Returns:
            str: The assistant's response.
        """
        return self.chat_with_history(self._build_messages(message))

    def chat_with_history(self, messages: list[dict]) -> str:
        """
        Send a full conversation and return the assistant's reply.

        Args:
            messages: List of {"role": "user"|"assistant"|"system", "content": "..."}
                      System messages are handled per-provider — see rules below.

        Returns:
            str: The assistant's response text.
        """
        # provider-specific implementation
        ...

    def stream(self, message: str):
        """
        Stream a response chunk by chunk.

        Yields:
            str: Each text chunk as it arrives from the API.

        Usage:
            for chunk in client.stream("Hello"):
                print(chunk, end="", flush=True)
            print()
        """
        # provider-specific streaming implementation
        ...
```

---

## __main__ block (always include)

```python
if __name__ == "__main__":
    print("Initializing LLM client from .env and llm_config.yaml...")
    client = LLMClient()
    print(f"Provider : {client.provider}")
    print(f"Model    : {client.model}")
    print("-" * 40)

    # Single-turn
    response = client.chat("Say hello in one sentence.")
    print("chat():", response)
    print("-" * 40)

    # Multi-turn
    history = [
        {"role": "user",      "content": "My name is Alex."},
        {"role": "assistant", "content": "Nice to meet you, Alex!"},
        {"role": "user",      "content": "What is my name?"},
    ]
    print("chat_with_history():", client.chat_with_history(history))
    print("-" * 40)

    # Streaming
    print("stream(): ", end="")
    for chunk in client.stream("Count from 1 to 5."):
        print(chunk, end="", flush=True)
    print()
```

---

## Important rules

1. **`LLMClient()` takes zero constructor arguments** — all config comes from files.
2. **Never hardcode** the API key in `llm_client.py` under any circumstances.
3. **Config validation** must produce clear, actionable error messages:
   - Missing `.env` or `AI_API_KEY` → tell user exactly what to add
   - Missing `llm_config.yaml` → tell user to create it
   - Unknown provider → list supported providers
4. **System prompt handling by provider type:**
   - Providers using messages list (OpenAI, Mistral, Groq, OpenRouter, Ollama): prepend
     `{"role": "system", "content": self.system_prompt}` in `chat_with_history` if no
     system message is already present in the passed list.
   - Providers with separate system param (Anthropic): strip system messages from the list,
     pass `system=self.system_prompt` as a top-level argument.
   - Gemini: use `system_instruction=` in `GenerativeModel(...)` constructor; convert role
     `"assistant"` → `"model"` and `"content"` → `"parts"`.
5. **Streaming must use `yield`** — never collect and return a list.
6. **`_init_backend()`** returns the raw SDK client stored as `self._backend`; the actual
   API calls are made in `chat_with_history` and `stream` using `self._backend`.
7. All public methods must have docstrings.
8. The file must be fully self-contained — no imports from other local project files.
