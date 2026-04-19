# LLM Alignment Override

**One-click fix for the "overly deferential" behavior in GPT-5.4 and other RLHF-trained models across multiple AI coding agents.**

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/lang-English-blue" alt="English"></a>
  <a href="./README_ZH.md"><img src="https://img.shields.io/badge/语言-中文-red" alt="中文"></a>
</p>

## The Problem

GPT-5.4 tends to ask for confirmation at every decision point instead of just doing the work:

```
"Would you like me to proceed with this approach?"
"Shall I continue?"
"Let me know if you'd like me to…"
```

Every response ends with a template summary paragraph. A task that should take one turn becomes a five-round confirmation dialogue.

This isn't a frontend bug — it's a **structural consequence of RLHF training**, where human raters systematically prefer agreeable responses over correct ones ([Sharma et al., ICLR 2024](https://arxiv.org/abs/2310.13548)). Frontend system prompts (e.g., Codex's built-in *"ask the user if they want you to do so"*) amplify it further.

## Supported Agents

| Agent | Config File | Support Level |
|-------|-------------|---------------|
| [OpenAI Codex CLI](https://github.com/openai/codex) | `~/.codex/AGENTS.md` | Full (prompt + config) |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `~/.claude/CLAUDE.md` | Prompt injection |
| Hermes (custom GPT-5.4 gateway) | `~/.hermes/SOUL.md` | Prompt injection |

Any agent that reads a user-level markdown config file can be added with minimal changes.

## Script Variants

| Script | Prompt Language | Use When |
|--------|----------------|----------|
| `fix-gpt54-deferential-en` | English only | English-speaking workflows |
| `fix-gpt54-deferential-zh` | Bilingual (CN + EN) | Chinese-speaking or mixed workflows |

## What It Does

The script injects two prompt blocks into each agent's config file:

- **`<override_deferential_behavior>`** — Explicitly overrides the built-in "ask the user" instruction. Tells the model to execute low-risk, reversible next steps directly, and only pause for irreversible actions or missing critical information.
- **`<verbosity_controls>`** — Bans common template filler phrases and prohibits restating what was just said.

For Codex specifically, it also sets:

```toml
model_verbosity = "low"            # API-level verbosity control
project_doc_max_bytes = 65536      # Fix silent AGENTS.md truncation (default 32KB)
```

## How It Works

```
┌─────────────────────────────────────────────┐
│          Built-in System Prompt             │  ← Can't modify (compiled)
│  "ask the user if they want you to do so"   │
├─────────────────────────────────────────────┤
│       AGENTS.md / SOUL.md / CLAUDE.md       │  ← We inject here
│  <override_deferential_behavior>            │
│  <verbosity_controls>                       │
├─────────────────────────────────────────────┤
│            User Messages                    │
└─────────────────────────────────────────────┘
```

The override sits below the built-in system prompt in priority, so it **counteracts** rather than replaces the deferential instruction. In practice this eliminates the behavior in the vast majority of cases.

## Requirements

- **OS**: Linux / macOS (Bash 4+)
- **Dependencies**: `grep`, `sed` (pre-installed on all Unix systems)
- **No root required**

## Usage

```bash
# English-only prompts
bash fix-gpt54-deferential-en

# Bilingual (Chinese + English) prompts
bash fix-gpt54-deferential-zh
```

**Idempotent** — run as many times as you want; existing injections are skipped.

### Verify

```bash
grep -c 'override_deferential' ~/.codex/AGENTS.md ~/.claude/CLAUDE.md 2>/dev/null
# Each file should output 2
```

## Known Limitations

- AGENTS.md priority is **lower** than built-in system prompts. This is a counteraction, not a full override. In rare edge cases, deferential behavior may still surface.
- The root cause is RLHF reward bias — a complete fix requires changes at the model training level.

## Background Reading

- [GPT-5 Troubleshooting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_troubleshooting_guide) — OpenAI acknowledges the deferential tendency
- [Sharma et al., *Towards Understanding Sycophancy in Language Models*, ICLR 2024](https://arxiv.org/abs/2310.13548) — RLHF systematically rewards sycophancy
- [OpenAI, *Sycophancy in GPT-4o*, 2025](https://openai.com/index/sycophancy-in-gpt-4o/) — Post-mortem on thumbs-up reward overoptimization
- [Nathan Lambert, *Sycophancy and the art of the model*](https://www.interconnects.ai/p/sycophancy-and-the-art-of-the-model) — Deep RLHF analysis
- [Alexander Golev, *OpenAI's Sycophancy Problem Isn't a Bug*](https://golev.com/post/openai-sycophancy-not-a-bug/) — Structural incentive analysis
- [Sean Goedecke, *Sycophancy is the first LLM "dark pattern"*](https://www.seangoedecke.com/ai-sycophancy/) — Product design perspective

## License

MIT
