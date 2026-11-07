# Claude Code 七层长任务 Harness

给 Zherui Li 的可复用工作区。这个仓库把 Claude Code 的项目说明、技能、权限、子代理、进度记录和完成条件放在一起，用来让 Opus 5.5 做完可以打开、可以核对、明天还能接着做的长任务。

A reusable workspace for Zherui Li. It keeps Claude Code project instructions, a skill, permissions, a reviewer subagent, a progress note, and a completion goal in one place, so an Opus 5.5 run can leave a file you can open, evidence you can check, and a handoff for the next session.

Harness 协调模型周围的说明、工具、权限、状态和检查。下面七层都使用 Claude Code 已有的配置位置。示例是文档工作流：参考材料在 `sources/`，草稿在 `drafts/`，定稿在 `published/`。

A harness coordinates the instructions, tools, permissions, state, and checks around the model. These seven layers use documented Claude Code features. The example is a document workflow: reference material in `sources/`, working files in `drafts/`, and approved files in `published/`.

灵感来自 [@beamnxw](https://x.com/beamnxw) 的文章 [Make Opus 5.5 Finish Long Tasks: A Reusable Harness Engineering Setup](https://x.com/beamnxw/status/2107522996046905797)，以及 [Anthropic Claude Code 文档](https://code.claude.com/docs)。

Inspired by [@beamnxw](https://x.com/beamnxw), [Make Opus 5.5 Finish Long Tasks: A Reusable Harness Engineering Setup](https://x.com/beamnxw/status/2107522996046905797), and the [Anthropic Claude Code docs](https://code.claude.com/docs).

## 七层 / The seven layers

| # | 层 | 落在哪里 |
| --- | --- | --- |
| 1 | 工作区事实：输出位置、来源要求和写作约定 | `CLAUDE.md` |
| 2 | 反复使用的写作与核验步骤 | `.claude/skills/write-draft/SKILL.md` |
| 3 | 任务材料：本地参考，以及可选的外部连接 | `sources/`，可选 Notion MCP |
| 4 | 执行边界：禁止读取密钥、禁止改写已发布文件 | `.claude/settings.json` |
| 5 | 只返回证据的审稿子代理 | `.claude/agents/evidence-reviewer.md` |
| 6 | 按工作分配推理力度：主会话 medium，审稿 high | 启动命令与子代理 `effort` |
| 7 | 用可检查的产物定义完成 | `/goal`，外加 `progress.md` 交接 |

| # | Layer | Where it lives |
| --- | --- | --- |
| 1 | Workspace facts: output locations, source rules, writing conventions | `CLAUDE.md` |
| 2 | The procedure you repeat: draft, review, save | `.claude/skills/write-draft/SKILL.md` |
| 3 | Source material: local files, plus an optional external connection | `sources/`, optional Notion MCP |
| 4 | Execution boundary: deny secret reads and edits to approved files | `.claude/settings.json` |
| 5 | A reviewer that returns evidence | `.claude/agents/evidence-reviewer.md` |
| 6 | Effort matched to the job: medium for the session, high for review | startup command and the subagent `effort` field |
| 7 | Completion defined by files and verification you can inspect | `/goal`, with `progress.md` for the next session |

## 目录 / Folders

- `sources/` — 参考材料和任务说明。先改 `sources/task.md`，再把官方原文或链接放进这个目录。
- `drafts/` — 进行中的稿件。`write-draft` 会写出 `drafts/article.md` 和 `drafts/checks.md`。
- `published/` — 已确认的文件。权限规则拒绝通过编辑工具改写这里的内容。发布需要单独的动作和授权。
- `progress.md` — 当前任务、已完成检查、决定、未决问题和下一步。下一轮会话从这里继续。

- `sources/` — reference material and the task brief. Fill `sources/task.md`, then add official sources or links here.
- `drafts/` — working files. `write-draft` saves `drafts/article.md` and `drafts/checks.md`.
- `published/` — approved files. The permission rule denies edits here through Claude Code's edit and write tools. Publishing stays a separate, authorized action.
- `progress.md` — the current task, finished checks, decisions, open issues, and the next action. The next session continues from this note.

## 怎么跑 / How to run

需要已安装 [Claude Code](https://code.claude.com/docs) 并登录账号。模型 ID 因账号而异，进会话后用 `/model` 确认当前模型。

Install [Claude Code](https://code.claude.com/docs) and sign in first. The model id can differ by account. After the session starts, confirm it with `/model`.

1. 填写 `sources/task.md`，并把参考材料放进 `sources/`。
2. 在仓库根目录启动：

   ```bash
   claude --model claude-opus-5-5 --effort medium
   ```

3. 材料在 Notion 时，可选添加 MCP，然后在会话里用 `/mcp` 完成认证，并先取回一个已知页面核对内容：

   ```bash
   claude mcp add --transport http notion https://mcp.notion.com/mcp
   ```

4. 第一次跑完整任务之前，检查已加载的说明、审稿代理和权限：`/context`、`/agents`、`/permissions`。
5. 设定完成条件。`/goal` 根据对话里展示的证据判断是否完成：

   ```text
   /goal Use write-draft to prepare an article from sources/. Completion requires drafts/article.md and drafts/checks.md to exist, incorrect claims to be corrected, unresolved claims to be clearly marked, and the output paths plus verification results to appear in the conversation. Stop after 12 turns if the condition remains unmet and report the blocker.
   ```

6. 结束后打开 `drafts/article.md`、`drafts/checks.md` 和 `progress.md`，抽几条主张核对来源。

1. Fill `sources/task.md` and put references in `sources/`.
2. From the repo root:

   ```bash
   claude --model claude-opus-5-5 --effort medium
   ```

3. Optional Notion MCP. Authenticate with `/mcp`, then retrieve one known page and confirm its contents before a long task depends on it:

   ```bash
   claude mcp add --transport http notion https://mcp.notion.com/mcp
   ```

4. Before the first full run: `/context`, `/agents`, `/permissions`.
5. Set the goal. `/goal` judges completion from evidence shown in the conversation:

   ```text
   /goal Use write-draft to prepare an article from sources/. Completion requires drafts/article.md and drafts/checks.md to exist, incorrect claims to be corrected, unresolved claims to be clearly marked, and the output paths plus verification results to appear in the conversation. Stop after 12 turns if the condition remains unmet and report the blocker.
   ```

6. When the run ends, open `drafts/article.md`, `drafts/checks.md`, and `progress.md`, and check a few claim-to-source matches yourself.

也可以直接调用技能：`/write-draft` 加上题目。`$ARGUMENTS` 会把题目传进同一套步骤。

You can also invoke the skill directly: `/write-draft` followed by the topic. `$ARGUMENTS` passes that text into the same procedure.

## 下一轮接着做 / Resume the next session

在同一工作区新开一次会话，粘贴：

Start a fresh session in this workspace and paste:

```text
Read progress.md and inspect the referenced draft and checks. Continue from the recorded next action and update the progress note.
```

`CLAUDE.md` 会在压缩上下文后重新读入。如果新会话从头重做，就补上进度笔记里缺的决定或文件路径。

Root `CLAUDE.md` is read again after compaction. If a new session starts over, add the missing decision or file path to the progress note.

## 用量 / Usage

用 `/usage` 看本次报告的用量，用 `/context` 看上下文里装了什么。把审稿和重试算进同一次任务。Anthropic 早期测试里提到的节省比例属于那次模型实验；这个仓库不把任何百分比当成你这次任务的结果。以你自己接受的成稿和 `/usage` 记录为准。

Use `/usage` for reported usage and `/context` for what occupies the working context. Count delegated review and retries in the same task. Figures from Anthropic's early-tester report belong to that model experiment. This repo does not treat any percentage as the result of your run. Measure completed work with `/usage`.

## 安全 / Secrets

`.claude/settings.json` 拒绝读取 `.env` 和 `.env.*`，并拒绝编辑 `published/**`。`.gitignore` 忽略 `.env`、`.env.*`、`.DS_Store`、`node_modules/` 和仅本地使用的 `.claude/settings.local.json`。项目级 `settings.json`、skills 和 agents 会纳入版本库。权限规则作用在 Claude Code 的工具边界上；用 `/permissions` 核对实际生效的规则。

`.claude/settings.json` denies `Read` of `.env` and `.env.*`, and denies `Edit` of `published/**`. `.gitignore` ignores `.env`, `.env.*`, `.DS_Store`, `node_modules/`, and the local-only `.claude/settings.local.json`. Project `settings.json`, skills, and agents stay tracked. Permission rules apply at Claude Code's tool boundary. Inspect the loaded rules with `/permissions`.
