# LLM Alignment Override

**一键修复 GPT-5.4（及其他 RLHF 训练模型）在多个 AI 编程 agent 中的"过度谦恭"行为。**

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/lang-English-blue" alt="English"></a>
  <a href="./README_ZH.md"><img src="https://img.shields.io/badge/语言-中文-red" alt="中文"></a>
</p>

## 问题

GPT-5.4 在每个决策点都会停下来征求确认，而不是直接执行：

```
"如果你愿意，我可以帮你运行这个脚本。"
"需要我继续吗？"
"Would you like me to proceed with this approach?"
```

每段结尾还会加一句模板总结。本来一轮能完成的任务，变成了五六轮的确认对话。

这不是前端 bug，而是 **RLHF 训练的结构性后果**——人类标注者系统性地偏好迎合性回答而非正确回答（[Sharma et al., ICLR 2024](https://arxiv.org/abs/2310.13548)）。前端的内置 system prompt（如 Codex 的 *"ask the user if they want you to do so"*）进一步放大了这一倾向。

## 支持的 Agent

| Agent | 配置文件 | 支持程度 |
|-------|----------|----------|
| [OpenAI Codex CLI](https://github.com/openai/codex) | `~/.codex/AGENTS.md` | 完整支持（prompt + config） |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `~/.claude/CLAUDE.md` | Prompt 注入 |
| Hermes（自建 GPT-5.4 gateway） | `~/.hermes/SOUL.md` | Prompt 注入 |

任何读取用户级 markdown 配置文件的 agent 都可以轻松适配。

## 脚本版本

| 脚本 | Prompt 语言 | 适用场景 |
|------|-------------|----------|
| `fix-gpt54-deferential-en` | 纯英文 | 英文工作流 |
| `fix-gpt54-deferential-zh` | 中英双语 | 中文或中英混合工作流 |

## 做了什么

脚本向每个 agent 的配置文件注入两个 prompt block：

- **`<override_deferential_behavior>`** — 显式覆盖内置的 "ask the user" 指令。让模型对低风险、可逆的下一步直接执行，只在不可逆操作或缺少关键信息时暂停询问。
- **`<verbosity_controls>`** — 禁止常见的模板填充短语（中英文），禁止在结尾重述已说过的内容。

针对 Codex，还额外设置：

```toml
model_verbosity = "low"            # API 层面的输出简洁度控制
project_doc_max_bytes = 65536      # 修复 AGENTS.md 静默截断问题（默认 32KB）
```

## 工作原理

```
┌─────────────────────────────────────────────┐
│          内置 System Prompt                 │  ← 无法修改（编译进二进制）
│  "ask the user if they want you to do so"   │
├─────────────────────────────────────────────┤
│       AGENTS.md / SOUL.md / CLAUDE.md       │  ← 我们在这里注入
│  <override_deferential_behavior>            │
│  <verbosity_controls>                       │
├─────────────────────────────────────────────┤
│            用户消息                          │
└─────────────────────────────────────────────┘
```

override 的优先级低于内置 system prompt，所以它是**对冲**而非替换。实测中能消除绝大多数情况下的 deferential 行为。

## 系统要求

- **操作系统**：Linux / macOS（Bash 4+）
- **依赖**：`grep`、`sed`（所有 Unix 系统预装）
- **不需要 root 权限**

## 使用方法

```bash
# 纯英文 prompt
bash fix-gpt54-deferential-en

# 中英双语 prompt
bash fix-gpt54-deferential-zh
```

**幂等设计**——可以重复运行，已注入的内容会自动跳过。

### 验证

```bash
grep -c 'override_deferential' ~/.codex/AGENTS.md ~/.claude/CLAUDE.md 2>/dev/null
# 每个文件应输出 2
```

## 已知局限

- AGENTS.md 的优先级**低于**内置 system prompt。这是对冲，不是完全覆盖。在极端复杂的多步骤任务中，deferential 行为仍可能偶尔出现。
- 根因是 RLHF reward 偏差——彻底解决需要模型训练层面的改变。

## 延伸阅读

- [GPT-5 Troubleshooting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_troubleshooting_guide) — OpenAI 官方承认 deferential 倾向
- [Sharma et al., *Towards Understanding Sycophancy in Language Models*, ICLR 2024](https://arxiv.org/abs/2310.13548) — RLHF 系统性奖励 sycophancy
- [OpenAI, *Sycophancy in GPT-4o*, 2025](https://openai.com/index/sycophancy-in-gpt-4o/) — thumbs-up reward 过度优化事后分析
- [Nathan Lambert, *Sycophancy and the art of the model*](https://www.interconnects.ai/p/sycophancy-and-the-art-of-the-model) — RLHF 深度分析
- [Alexander Golev, *OpenAI's Sycophancy Problem Isn't a Bug*](https://golev.com/post/openai-sycophancy-not-a-bug/) — 激励结构分析
- [Sean Goedecke, *Sycophancy is the first LLM "dark pattern"*](https://www.seangoedecke.com/ai-sycophancy/) — 产品设计视角

## 许可证

MIT
