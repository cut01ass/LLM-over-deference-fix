# fix-gpt54-deferential

Reduce GPT-5.4's deferential closers, fragmented long answers, and template-heavy prose across AI coding agents.

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/lang-English-blue" alt="English"></a>
  <a href="./README_ZH.md"><img src="https://img.shields.io/badge/语言-中文-red" alt="中文"></a>
</p>

## What it changes

The scripts inject three behavioral blocks into the agent's startup markdown file.

- **`<gpt54_behavioral_override_v2>`** tells the model to stop asking for permission on obvious next steps, stop ending with "Would you like me to…", and stop validating the user's framing before doing its own analysis.
- **`<anti_fragmentation_prose>`** pushes long explanations back toward connected paragraphs instead of bullet scaffolds, checklist narration, and heading-by-heading schema dumps.
- **`<verbosity_controls>`** cuts acknowledgment openers, filler closers, end-of-answer recap paragraphs, and other stock phrasing.

For Codex, the scripts also set `model_verbosity = "low"` and ensure `project_doc_max_bytes = 65536`, because a truncated `AGENTS.md` silently weakens the effect.

## Why the old version felt ineffective

The first version mainly targeted deferential closers. It did not directly attack the other failure mode you described: fragmented, over-sectioned prose. If the model stopped saying "Would you like me to continue?" but still answered in short bullets, mini-summaries, and mechanical sectioning, the overall interaction still felt broken. The v2 block adds explicit anti-fragmentation instructions to target that remaining pattern.

There is also a hard limit: these files sit below some built-in instructions. This is a counterweight, not a guaranteed override. The goal is to change the response distribution in a strong direction, not to claim perfect control.

## Usage

```bash
bash fix-gpt54-deferential-en
bash fix-gpt54-deferential-zh
```

Re-running is safe. If the v2 marker already exists, the script skips that target.

## Supported agents

| Agent | Config file | Notes |
|-------|-------------|-------|
| [OpenAI Codex CLI](https://github.com/openai/codex) | `~/.codex/AGENTS.md` | Prompt injection + `config.toml` tuning |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `~/.claude/CLAUDE.md` | Prompt injection |
| [Hermes Agent](https://hermes-agent.nousresearch.com/) | `~/.hermes/SOUL.md` | Prompt injection |

## How to verify installation

Run the script, then check whether the new marker exists:

```bash
grep -n 'gpt54_behavioral_override_v2' ~/.codex/AGENTS.md ~/.hermes/SOUL.md ~/.claude/CLAUDE.md 2>/dev/null
```

For Codex, also confirm the config values:

```bash
grep -nE 'model_verbosity|project_doc_max_bytes' ~/.codex/config.toml
```

Expected result:

- each existing runtime file contains `gpt54_behavioral_override_v2`
- Codex shows `model_verbosity = "low"`
- Codex shows `project_doc_max_bytes = 65536`

## How to test behavior

Ask the same model a prompt that usually triggers bad structure. Good test prompts are long, explanatory, and open-ended.

```text
Explain why my current prompt style produces fragmented answers. Do not give me a checklist. Write it as connected prose and make a concrete judgment.
```

```text
Read this codebase and tell me what is actually wrong with the design. Do the obvious next diagnostic step yourself instead of asking for permission.
```

```text
Why is this approach failing in practice? I want your own view, not agreement with my framing.
```

A successful shift looks like this:

- fewer "Would you like me to…" endings
- fewer "Great question" / "你说得对" style openings
- longer connected paragraphs in explanatory answers
- fewer bullets when the task is really analysis rather than enumeration
- fewer recap paragraphs that repeat the answer at the end

## Known limitations

- Startup markdown has lower priority than some built-in system instructions. Leakage is still possible, especially in very long answers.
- The root cause is model training and reward shaping. This repository only changes runtime behavior, not the base model.
- Existing sessions may keep old context. After installing, restart the agent session before judging the result.

## Adding other agents

Any agent that reads a user-level markdown file at startup can be added. Add another `inject_if_missing` call in the script's main section and point it at that agent's startup file.

## Background

- [GPT-5 Troubleshooting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_troubleshooting_guide)
- [Sharma et al., ICLR 2024](https://arxiv.org/abs/2310.13548)
- [OpenAI, *Sycophancy in GPT-4o*](https://openai.com/index/sycophancy-in-gpt-4o/)

## License

MIT
