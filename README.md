# Baton — Multi-Agent Handoff Protocol

> A structured interchange format and CLI for passing context between AI agents across providers.

---

## What is Baton?

Baton is an open protocol and Python library for creating **structured handoff documents** between AI agents. When you work across multiple AI systems (e.g., Claude, OpenAI, local models), context gets lost at every boundary. Baton solves this by packaging conversation history, decisions, artifacts, and task state into a portable document that any AI can consume.

The primary output is human-readable Markdown — paste it directly into any AI chat to resume where you left off.

---

## Architecture

```
Your Orchestrator / Agent
        |
        v
integrations/anthropic_client.py   ← Claude Messages API bridge
integrations/openai_client.py      ← OpenAI bridge (planned)
        |
        v
baton/
  parser.py       ← Parse raw conversation transcripts
  compiler.py     ← Extract structured context (heuristics-based)
  models.py       ← Pydantic data models (BatonDocument, Conversation, etc.)
  redaction.py    ← Privacy / confidentiality layer
  validation.py   ← Schema + integrity checks
  cli.py          ← Command-line interface
```

---

## Installation

```bash
pip install -e .
```

Or with pip directly once published:

```bash
pip install baton-messages
```

**Requirements:**
- Python 3.10+
- `ANTHROPIC_API_KEY` environment variable (for Claude integration)
- `OPENAI_API_KEY` environment variable (for OpenAI integration)

Set your API key:

```bash
export ANTHROPIC_API_KEY=your-key-here
```

---

## CLI Usage

```bash
# Parse a conversation file into structured output
baton parse inputs/transcript.txt

# Generate a handoff document
baton handoff samples/conversation.txt --format markdown

# Control output size
baton handoff samples/conversation.txt --size compact     # < 2K tokens
baton handoff samples/conversation.txt --size standard    # < 8K tokens
baton handoff samples/conversation.txt --size full        # everything

# Apply a redaction policy
baton handoff samples/conversation.txt --redact policy.yaml

# Validate a handoff document
baton validate handoff.json

# Show metadata and token estimates
baton info handoff.json

# Pipe mode (stdin → stdout)
cat transcript.txt | baton pipe
```

---

## Python API

```python
import baton

# Parse a raw conversation
conversation = baton.parse(text)

# Compile into a structured handoff document
document = baton.compile(conversation, size="standard")

# Export to Markdown (paste into any AI chat)
markdown = baton.export(document, format="markdown")

# Apply redaction before sharing
clean_doc = baton.redact(document, policy)

# Validate integrity
result = baton.validate(document)
```

---

## The BatonDocument Format

A handoff document contains these sections:

| Section | Description |
|---|---|
| `context` | What this work is about |
| `decisions` | Things that have been decided |
| `artifacts` | Code, files, and outputs produced |
| `state` | Current status of the work |
| `task` | What the receiving agent should do next |
| `open_questions` | Unresolved items |
| `constraints` | Rules the receiving agent must respect |
| `metadata` | Source model, timestamp, token count, baton version |

Serializes to both **Markdown** (human-readable, paste anywhere) and **JSON** (machine-parseable).

---

## Claude Integration

The `integrations/anthropic_client.py` module wraps the Anthropic Messages API:

```python
import anthropic
import os

client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=4096,
    system="You are a Python library developer...",
    messages=[{"role": "user", "content": "Your prompt here"}],
)
```

No credentials are hardcoded. Always pass keys via environment variables or a secrets manager.

---

## Milestones

- [x] **Milestone 1** — BatonDocument format spec + Pydantic models
- [x] **Milestone 2** — Conversation parser (auto-detects Claude, ChatGPT, plain text, JSON, Markdown)
- [x] **Milestone 3** — Context compiler (heuristics-based: decisions, artifacts, open questions, state)
- [x] **Milestone 4** — Provider format translators (import/export Anthropic, OpenAI, universal Markdown)
- [x] **Milestone 5** — Trust and redaction layer (RedactionPolicy, PII scanning, hash-based integrity)
- [x] **Milestone 6** — CLI (`parse`, `handoff`, `validate`, `pipe`, `info`)
- [x] **Milestone 7** — Python library (public API, fully typed, documented)
- [ ] **Milestone 8** — Demo + README (end-to-end walkthrough, BATON-SPEC.md)

---

## Redaction

Apply a YAML policy to control what gets shared:

```yaml
# policy.yaml
decisions: share
artifacts: share
conversation: redact
open_questions: summarize
```

The redaction layer also auto-flags potential PII and API keys before export.

---

## Repository Structure

```
README.md
BATON-SPEC.md              # Full protocol specification
CHANGELOG.md
pyproject.toml
requirements.txt
baton/
  __init__.py
  cli.py
  models.py
  compiler.py
  parser.py
  redaction.py
  validation.py
integrations/
  anthropic_client.py
samples/                   # Sample transcripts for testing
demo/                      # End-to-end demo handoff documents
```

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | Yes (for Claude) | Your Anthropic API key |
| `OPENAI_API_KEY` | Planned | Your OpenAI API key |

**Never commit API keys to version control.** Add `.env` to your `.gitignore`.

---

## .gitignore

Make sure your repo includes:

```
.env
*.env
__pycache__/
*.pyc
dist/
*.egg-info/
.DS_Store
```

---

## Contributing

This project is framed as an open protocol proposal — not just a tool. If you're building multi-agent systems and want a common interchange format, contributions and feedback are welcome.

---

## License

MIT

