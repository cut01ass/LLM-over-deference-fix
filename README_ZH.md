# fix-gpt54-deferential

修复 GPT-5.4 在 AI 编程 agent 中的"过度谦恭"行为。

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/lang-English-blue" alt="English"></a>
  <a href="./README_ZH.md"><img src="https://img.shields.io/badge/语言-中文-red" alt="中文"></a>
</p>

## 做了什么

向 agent 配置文件注入两个 prompt block，对冲 GPT-5.4 的 RLHF sycophancy：

- **`<override_deferential_behavior>`** — 原则性身份声明（"prioritize accuracy over validation"）+ 显式覆盖内置的 "ask the user" 指令。消除"如果你愿意…" / "Would you like me to…" 式结尾，促进对错误前提的 pushback。
- **`<verbosity_controls>`** — 强制以实质内容结尾，禁止 acknowledgment 开头语（"好问题！"）、模板填充、机械枚举、结尾重述。

针对 Codex 额外设置 `model_verbosity = "low"` 并提升 `project_doc_max_bytes` 到 65536（修复 AGENTS.md 静默截断）。

## 用法

```bash
bash fix-gpt54-deferential-en    # 纯英文 prompt
bash fix-gpt54-deferential-zh    # 中英双语 prompt
```

幂等设计——重复运行会跳过已注入的内容。

## 支持的 agent

| Agent | 配置文件 | 说明 |
|-------|----------|------|
| [OpenAI Codex CLI](https://github.com/openai/codex) | `~/.codex/AGENTS.md` | Prompt + config.toml 修复 |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `~/.claude/CLAUDE.md` | Prompt 注入 |
| [Hermes Agent](https://hermes-agent.nousresearch.com/) | `~/.hermes/SOUL.md` | Prompt 注入 |

## 系统要求

仅支持 Linux。需要 `bash`、`grep`、`sed`（预装）。不需要 root。

## 适配其他 agent

任何启动时读取用户级 markdown 文件的 agent 都可以适配：

1. 找到 agent 读配置的路径（如 `~/.cursor/rules/`、`~/.continue/config.md`）。
2. 在脚本的 `# --- main ---` 部分加一行：
   ```bash
   inject_if_missing "$HOME/.your-agent/CONFIG.md" "Your Agent"
   ```
3. Prompt block 本身是 agent 无关的——任何处理 markdown 指令的模型都能用。

## 已知局限

- 配置文件优先级低于内置 system prompt，是**对冲**而非完全覆盖——长回答中偶尔仍可能泄漏。
- 根因是 RLHF reward 偏差，彻底解决需要模型训练层面的改变。

## 延伸阅读

- [GPT-5 Troubleshooting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_troubleshooting_guide) — OpenAI 官方承认 deferential 倾向
- [Sharma et al., ICLR 2024](https://arxiv.org/abs/2310.13548) — RLHF 系统性奖励 sycophancy
- [OpenAI, *Sycophancy in GPT-4o*](https://openai.com/index/sycophancy-in-gpt-4o/) — thumbs-up reward 过度优化事后分析

## 许可证

MIT
