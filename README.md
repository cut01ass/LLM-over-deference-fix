# fix-gpt54-deferential

Fix GPT-5.4's "overly deferential" behavior across AI coding agents.

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/lang-English-blue" alt="English"></a>
  <a href="./README_ZH.md"><img src="https://img.shields.io/badge/语言-中文-red" alt="中文"></a>
</p>

## What it does

Injects two prompt blocks into your agent's config file to counteract GPT-5.4's RLHF-trained sycophancy:

- **`<override_deferential_behavior>`** — Principle-based identity framing ("prioritize accuracy over validation") + explicit override of the built-in "ask the user" instruction. Eliminates "Would you like me to…" / "如果你愿意…" closers, promotes pushback on incorrect premises.
- **`<verbosity_controls>`** — Enforces substantive endings, bans acknowledgment preambles ("Great question!"), template filler, mechanical enumeration, summary paragraphs.

For Codex, also sets `model_verbosity = "low"` and raises `project_doc_max_bytes` to 65536 (fixes silent AGENTS.md truncation).

## Usage

```bash
bash fix-gpt54-deferential-en    # English-only prompts
bash fix-gpt54-deferential-zh    # Bilingual (Chinese + English) prompts
```

Idempotent — re-running skips existing injections.

## Supported agents

| Agent | Config file | Notes |
|-------|-------------|-------|
| [OpenAI Codex CLI](https://github.com/openai/codex) | `~/.codex/AGENTS.md` | Prompt + config.toml fixes |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `~/.claude/CLAUDE.md` | Prompt injection |
| [Hermes Agent](https://hermes-agent.nousresearch.com/) | `~/.hermes/SOUL.md` | Prompt injection |

## Requirements

Linux only. Needs `bash`, `grep`, `sed` (pre-installed). No root required.

## Adding other agents

Any agent that reads a user-level markdown file at startup can be added. The pattern is:

1. Find where the agent reads its config (e.g., `~/.cursor/rules/`, `~/.continue/config.md`).
2. Add a new block in the script's `# --- main ---` section:
   ```bash
   inject_if_missing "$HOME/.your-agent/CONFIG.md" "Your Agent"
   ```
3. That's it. The prompt blocks are agent-agnostic — they work with any model that processes markdown instructions.

## Known limitations

- Config file priority is lower than built-in system prompts. This is a **counteraction**, not a full override — edge-case leakage is possible in long responses.
- Root cause is RLHF reward bias. A complete fix requires model training changes.

## Background

- [GPT-5 Troubleshooting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_troubleshooting_guide) — OpenAI acknowledges the tendency
- [Sharma et al., ICLR 2024](https://arxiv.org/abs/2310.13548) — RLHF systematically rewards sycophancy
- [OpenAI, *Sycophancy in GPT-4o*](https://openai.com/index/sycophancy-in-gpt-4o/) — Thumbs-up reward overoptimization post-mortem

## License

MIT
