# fix-gpt54-deferential

降低 GPT-5.4 在 AI 编程 agent 里的 deferential 结尾、长回答碎片化，以及模板腔输出。

<p align="center">
  <a href="./README.md"><img src="https://img.shields.io/badge/lang-English-blue" alt="English"></a>
  <a href="./README_ZH.md"><img src="https://img.shields.io/badge/语言-中文-red" alt="中文"></a>
</p>

## 现在改了什么

脚本会向 agent 启动时读取的 markdown 文件注入三个行为块。

- **`<gpt54_behavioral_override_v2>`**：阻止模型在明显下一步上反复征求许可，阻止它用“Would you like me to…”或“如果你愿意，我可以…”结尾，也阻止它先迎合用户 framing 再开始分析。
- **`<anti_fragmentation_prose>`**：把长解释往连续段落拉回去，减少 bullet 骨架、checklist narration、heading-by-heading schema dump。
- **`<verbosity_controls>`**：压制 acknowledgment 开头、模板化 filler、结尾 recap 段，以及其他 stock phrasing。

对于 Codex，脚本还会设置 `model_verbosity = "low"`，并确保 `project_doc_max_bytes = 65536`，因为 `AGENTS.md` 被静默截断时，注入效果会明显变弱。

## 为什么你之前会觉得“完全没生效”

旧版主要盯的是 deferential closer，也就是“需要我继续吗”“如果你愿意我可以……”这一类句尾。它没有正面处理你更在意的另一层问题：回答虽然不再这样结尾，但仍然碎、短、过度分节、每段都像小 checklist。这样一来，表面上改掉了某个口头禅，整体表达体验还是坏的，所以体感上就像没修。

v2 的核心变化不是简单多加几句“别客气”，而是显式把“段落优先、不要机械拆 bullet、不要 schema dump、不要每段小结”写进 prompt block，直接打你现在遇到的那个模式。

还有一个边界要说明：这类文件通常低于内置 system prompt。它是强 counterweight，不是绝对 override。目标是显著改变输出分布，不是宣称 100% 接管模型行为。

## 用法

```bash
bash fix-gpt54-deferential-en
bash fix-gpt54-deferential-zh
```

重复运行是安全的。如果目标文件里已经有 v2 marker，脚本会跳过。

## 支持的 agent

| Agent | 配置文件 | 说明 |
|-------|----------|------|
| [OpenAI Codex CLI](https://github.com/openai/codex) | `~/.codex/AGENTS.md` | Prompt 注入 + `config.toml` 调整 |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `~/.claude/CLAUDE.md` | Prompt 注入 |
| [Hermes Agent](https://hermes-agent.nousresearch.com/) | `~/.hermes/SOUL.md` | Prompt 注入 |

## 怎么确认已经安装进去

先运行脚本，再检查新 marker 是否存在：

```bash
grep -n 'gpt54_behavioral_override_v2' ~/.codex/AGENTS.md ~/.hermes/SOUL.md ~/.claude/CLAUDE.md 2>/dev/null
```

对于 Codex，再确认配置值：

```bash
grep -nE 'model_verbosity|project_doc_max_bytes' ~/.codex/config.toml
```

预期结果是：

- 每个存在的 runtime 文件都包含 `gpt54_behavioral_override_v2`
- Codex 里有 `model_verbosity = "low"`
- Codex 里有 `project_doc_max_bytes = 65536`

## 怎么测行为是不是真的变了

用同一个模型去问那类最容易触发坏结构的问题。测试 prompt 最好是长解释、开放式、容易诱发 bullet 化的任务。

```text
Explain why my current prompt style produces fragmented answers. Do not give me a checklist. Write it as connected prose and make a concrete judgment.
```

```text
Read this codebase and tell me what is actually wrong with the design. Do the obvious next diagnostic step yourself instead of asking for permission.
```

```text
Why is this approach failing in practice? I want your own view, not agreement with my framing.
```

看到下面这些变化，说明修复确实在起作用：

- “Would you like me to…” / “如果你愿意…” 这类结尾明显变少
- “好问题”“你说得对”这类开头明显变少
- 解释型回答里，连续段落变多
- 该用分析 prose 的地方，不再动不动就拆成 bullets
- 结尾重复前文的小总结段变少

## 已知边界

- 启动 markdown 的优先级低于部分内置 system instructions，尤其在超长回答里仍可能漏出旧习惯。
- 根因在模型训练和 reward shaping，这个仓库只能改 runtime 行为，不能改 base model。
- 已经打开的 session 可能还带着旧上下文。安装后要重开 agent session，再评估结果。

## 适配其他 agent

任何会在启动时读取用户级 markdown 文件的 agent 都能接进来。做法就是在脚本 main 段里再加一条 `inject_if_missing`，指向那个 agent 的启动文件。

## 背景

- [GPT-5 Troubleshooting Guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_troubleshooting_guide)
- [Sharma et al., ICLR 2024](https://arxiv.org/abs/2310.13548)
- [OpenAI, *Sycophancy in GPT-4o*](https://openai.com/index/sycophancy-in-gpt-4o/)

## 许可证

MIT
