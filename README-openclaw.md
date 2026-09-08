# OpenClaw / 云养虾（ArkClaw）适配说明

本仓库（awesome-novel-agent）已适配 **OpenClaw**（俗称"小龙虾"）及其云端版本 **云养虾 ArkClaw**，
可在手机 / 电脑网页端直接使用完整的多 Agent 小说写作流水线（设定 → 卷纲 → 章纲 → 提示词 → 正文 → 去AI味 → 评审 → 归档）。

## 原理

OpenClaw 的技能格式（目录 + `SKILL.md`，frontmatter 只需 `name` + `description`）与 Claude Code 同源，
且 OpenClaw 支持子代理（`sessions_spawn`）。适配方式与仓库已有的 ZCode/Reasonix 平台同构：

| 部件 | 部署位置 | 说明 |
|---|---|---|
| 入口 skill（awesome-novel） | `~/.openclaw/skills/awesome-novel/`（用户级） | 检测项目状态、初始化、迁移 |
| 9 个子代理 + 2 个工具 | 项目内 `.openclaw/skills/<name>/SKILL.md`（11 个） | agents 即 skills |
| 反 AI 规则 / 文风 / 格式 / 题材 | 项目内 `.openclaw/knowledge/` | init.py 按题材裁剪合并 |
| 动态写作记忆 | 项目内 `.openclaw/memory/` | 各环节作者反馈持续积累 |
| 子代理身份说明书 | 项目根 `AGENTS.md` | OpenClaw 子代理上下文只注入它，是关键通道 |

**调度协议：** novel-agent 写 order 到 `.agent/task/*-order.md`（`status: pending`）
→ `sessions_spawn` 派发（task 首行 `[Subagent Task]`，含操作手册路径 + order 路径）
→ 子代理先 Read 自己的 SKILL.md 再执行 → 覆盖 order 为 `status: DONE` → novel-agent 用 `sessions_yield` 等待完成事件（推送式，不轮询）。

## 安装（云养虾 ArkClaw / OpenClaw）

### 方式一：让 AI 自己装
在 OpenClaw / 云养虾对话里说：
> **帮我安装 awesome-novel-skill，仓库在 https://github.com/d1667018881/awesome-novel-agent-openclaw**

AI 会克隆仓库并运行 `./install.sh openclaw`，把技能装到 `~/.openclaw/skills/awesome-novel/`。

### 方式二：手动安装（本地 OpenClaw）
```bash
git clone https://github.com/d1667018881/awesome-novel-agent-openclaw.git
cd awesome-novel-agent && ./install.sh openclaw
```

## 初始化小说项目

在 OpenClaw 中打开目标目录（云养虾里即你的云盘工作目录），说：
> **帮我写本小说**

skill 会先询问确认，然后自动运行初始化；也可手动：
```bash
python ~/.openclaw/skills/awesome-novel/tools/init.py <小说项目路径> --genre <编号> --platform openclaw
```
（`--genre` 可省略，交互式选题材，1-24 共 24 个题材）

## 开始写作

初始化完成后，在项目目录说 **"帮我写本小说"** 或 **"帮我继续写"**，主代理加载
`AGENTS.md` 与 `.openclaw/skills/novel-agent/SKILL.md` 扮演总指挥，进入写作循环。
每章写完会问你是否继续下一章。

## 升级

```bash
python ~/.openclaw/skills/awesome-novel/tools/sync-project.py <小说项目路径> --platform openclaw
```

## 注意事项

1. **子代理 skill 不依赖自动发现**：子代理被 `sessions_spawn` 后主动 Read 自己的 SKILL.md，
   无需配置 `skills.load.extraDirs`。
2. **pyyaml 依赖**：openclaw 平台的 agent→skill 转换需要 pyyaml（`pip install pyyaml`）。
3. **手机端使用**：云养虾是网页端，直接打开浏览器即可；小说项目文件在你的云工作目录里，
   技能与项目随云盘同步，换设备不丢进度。
