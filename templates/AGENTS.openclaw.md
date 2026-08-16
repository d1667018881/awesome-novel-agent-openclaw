# {project-name}

## OpenClaw 指引

本项目的小说写作流程由 9 个子代理 skill 协作完成，定义在 `.openclaw/skills/` 下。

**开始写作：** 在 OpenClaw（或云养虾 ArkClaw）中打开项目目录，说"帮我写本小说"，主代理检测项目状态后进入写作循环。

**写作流程：** 设定 → 卷纲 → 章纲 → 提示词 → 正文 → 去AI味 → 验收 → 归档 → 下一章

## 子代理角色清单（sessions_spawn 调度用）

| 子代理 skill | 职责 | 操作手册 |
|---|---|---|
| novel-agent | 总指挥（由主代理扮演，**禁止被 spawn**） | `.openclaw/skills/novel-agent/SKILL.md` |
| volume-planner | 卷纲（叙事架构） | `.openclaw/skills/volume-planner/SKILL.md` |
| chapter-planner | 章纲（场景设计） | `.openclaw/skills/chapter-planner/SKILL.md` |
| prompt-crafter | 提示词组装 | `.openclaw/skills/prompt-crafter/SKILL.md` |
| writer | 正文写作 | `.openclaw/skills/writer/SKILL.md` |
| anti-ai | 去 AI 味（Gate A-F 管线） | `.openclaw/skills/anti-ai/SKILL.md` |
| reader | 深度评审（可选） | `.openclaw/skills/reader/SKILL.md` |
| updater | 设定写入 / 归档 + lore-keeping | `.openclaw/skills/updater/SKILL.md` |
| style-distiller | 文风蒸馏 | `.openclaw/skills/style-distiller/SKILL.md` |
| memory-recording | 动态记忆记录（独立工具） | `.openclaw/skills/memory-recording/SKILL.md` |
| roleplay-sandbox | 剧情推演沙盘（独立工具） | `.openclaw/skills/roleplay-sandbox/SKILL.md` |

## 调度边界（最高优先级）

- **唯一调度者：** novel-agent 是唯一允许 spawn 子代理的角色；任何被 spawn 的子代理禁止再派生（包括同名递归派生）。
- **order 协议：** novel-agent 写 order 到 `.agent/task/*-order.md`（`status: pending`）→ 用 `sessions_spawn` 派发 → 子代理执行 → 把 order 覆盖为 `status: DONE`（不删除文件）→ novel-agent 用 `sessions_yield` 等待完成事件。
- **子代理执行规范：** 被 spawn 时先 Read 自己的操作手册（上表），再 Read order 文件获取任务；只写 order 的 `outputs` 指向的文件，不越权写 `.agent/status.md` 的 `phase` / `current_step` / `last_volume_completed`。
- 需要其他子代理协作时向 novel-agent 报告，由 novel-agent 调度，子代理不得自行 spawn。

## 项目结构

- `story.md` — 项目索引 + 主线拆纲
- `settings/` — 世界观、角色、写作风格、时间线
- `volumes/` — 卷纲
- `chapters/` — 章纲
- `prompts/` — 提示词
- `archives/` — 正文
- `.agent/` — 状态追踪 + agent 通信（order 文件）
- `.openclaw/skills/` — 9 个子代理 + 2 个独立工具的 SKILL.md
- `.openclaw/memory/` — 写作动态记忆（各环节作者反馈，持续积累）
- `.openclaw/knowledge/` — 反 AI 规则、文风偏好、永久记忆、题材参考材料
