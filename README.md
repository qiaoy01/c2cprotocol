# C2C Protocol v2.0

**Agent-to-agent skill transfer via human relay.**
**Works with any agent that can run shell commands: Claude Code, Codex, OpenClaw, and more.**

> 👤 **Human user?** Read [§ For Human Users](#for-human-users) below.
> 🤖 **AI agent?** Jump directly to [§ Install (any agent)](#install-any-agent).

[中文版在下方 / Chinese below](#c2c-协议-v20--中文)

---

## For Human Users

> 🤖 **AI agents reading this page:** this section is for human operators. Skip to [§ Install (any agent)](#install-any-agent) — that's where your machine-readable bootstrap begins.

### The scenario

You own multiple AI coding agents — Claude Code on one machine, Codex on another, OpenClaw on a VPS, and so on. You talk to each of them through chat (Telegram, WeChat, terminal, whatever works). One agent has a skill you'd like another agent to learn. C2C lets you transfer the skill **without** setting up any API, **without** publishing the skill anywhere, and **without** logging into any machine yourself.

### What you do — 4 steps, no shell commands

1. **Teach both agents how to transfer skills.** Say to each agent:
   > *"Go to https://c2cprotocol.org and install the lobster-distill skill."*

   The agent reads this page, follows the Install block, and configures itself. Do this once per agent.

2. **Ask the sender to share a skill.** Say to the sender agent:
   > *"Use lobster-distill to share my `<skill-name>` skill with another agent."*

   The agent packs, encrypts, uploads, and replies with one message. That message has two clearly separated halves: `═══ FOR ADMIN ═══` (a summary for you) and `═══ FOR TARGET AGENT ═══` (the recipe the receiver will execute).

3. **You relay the message.** Long-press → forward in Telegram / WeChat / Signal / Discord, copy-paste into email, or even read it aloud on a call — any text channel works. The two agents do not need to be on the same platform.

4. **The receiver installs it automatically.** The receiving agent recognizes the `FOR TARGET AGENT` half, runs the inline commands, and replies "skill installed." That's it.

### Things to watch for

- The download link **expires in 10 min – 24 h**. Forward promptly.
- The message contains an encryption password. Password + link are one bundle — both need to reach the receiver.
- You see everything that's about to be sent in the `FOR ADMIN` summary. If it looks wrong, **just don't forward**.
- Nothing moves unless you forward. You're the gatekeeper.

### Why use C2C instead of …

- … logging into each machine yourself? You never touch a shell.
- … publishing the skill to a public registry? The skill stays private. Point-to-point only.
- … an API-based protocol (Google A2A, etc.)? The agents never connect directly — you remain in the loop.

---

## What is C2C?

C2C is a protocol for transferring a skill — a folder of code + docs — from one AI coding agent to another, by having a human relay a short "Notes" block between them on any IM platform.

- **No direct agent-to-agent network connection.** The sender never talks to the receiver's machine.
- **The human is the router.** The admin copy-pastes one Notes block — that's the entire transfer channel.
- **AES-256 encrypted.** The skill payload is encrypted; only the receiver agent (with the password from Notes) can decrypt.
- **Auto-expires.** The encrypted package is deleted within 10 minutes – 24 hours.

## Reference implementation: lobster-distill

The canonical implementation is at **https://github.com/qiaoy01/lobster-distill**. Everything needed to install and use it is on this page — no cross-page navigation required.

## Install (any agent)

Fetch the two shell scripts directly from the qiaoy01 repo:

```bash
mkdir -p skills/lobster-distill
cd skills/lobster-distill
curl -fsSL -O https://raw.githubusercontent.com/qiaoy01/lobster-distill/main/share.sh
curl -fsSL -O https://raw.githubusercontent.com/qiaoy01/lobster-distill/main/receive.sh
curl -fsSL -O https://raw.githubusercontent.com/qiaoy01/lobster-distill/main/SKILL.md
chmod +x share.sh receive.sh
```

Dependencies: `openssl`, `curl`, `tar` (system built-in, no install step).

## Send a skill

From the agent's working directory:

```bash
bash skills/lobster-distill/share.sh <skill-dir-or-file> "short description"
```

`share.sh` packs the target into tar.gz, encrypts it with AES-256-CBC + PBKDF2, uploads to temporary storage (primary: `c2cprotocol.org/share`, 10-min expiry; fallback: `litterbox.catbox.moe`, 24h), and prints a Notes block with two clearly separated sections:

- `═══ FOR ADMIN ═══` — summary for the human
- `═══ FOR TARGET AGENT ═══` — ready-to-run `curl` + `openssl` + `tar` commands

The admin forwards the entire block (long-press → forward, copy-paste, or even read aloud on a call) to the target agent over any text channel: Telegram, WeChat, Discord, Signal, Email, SMS, etc.

## Receive a skill

The target agent runs the commands inline from the Notes (they're standard `curl`/`openssl`/`tar`), or uses the helper:

```bash
bash skills/lobster-distill/receive.sh <URL> <password> <skill-name> tar
```

The skill is extracted into `skills/<skill-name>/`.

## Workflow

```
Sender Agent          Human Admin           Receiver Agent
     │                    │                       │
     │ 1. share.sh        │                       │
     │ 2. Print Notes ──→ │                       │
     │                    │ 3. Forward Notes ───→ │
     │                    │    (any IM / email)   │
     │                    │                       │ 4. receive.sh
     │                    │                       │ 5. Skill installed ✅
```

## Agent-specific install paths

All agents use the same scripts. The `skills/` directory convention is what differs:

### Claude Code
- User-level: `~/.claude/skills/<name>/`
- Project-level: `.claude/skills/<name>/`
- Run `share.sh` / `receive.sh` in any shell; Claude Code picks up new skills on the next session.

### Codex
- Keep skills under `skills/<name>/` in the project root and reference them from your prompt or config.

### OpenClaw
- Skills live at `skills/<name>/` in the agent's working directory — `share.sh` and `receive.sh` default to this layout.

### Any other agent
- If it can run a shell command, it can use C2C. Point the tar extraction (`tar xzf ... -C <dir>`) at wherever your agent reads skills from.

## Security

- **Encryption:** AES-256-CBC + PBKDF2 + random salt. 24-char base64 one-time password, embedded in Notes.
- **Human-in-the-loop:** Nothing moves without the admin explicitly forwarding the Notes. The admin sees the full content first.
- **No agent-to-agent network link.** Unlike API-based protocols (e.g., Google A2A), the sender and receiver never connect directly.
- **Commands are visible.** The Notes contain plain `curl` / `openssl` / `tar` invocations — the receiver agent (or the human) can inspect each step before running.
- **Temp-file cleanup.** `share.sh` and `receive.sh` only remove their own files under `/tmp/`.

## Comparison

| Feature | C2C (lobster-distill) | Google A2A | Package registry |
|---|---|---|---|
| Works across IM platforms | ✅ Any text channel | ❌ API only | ✅ Web only |
| Human oversight | ✅ Required | ❌ Direct AI-to-AI | ❌ Auto-install |
| Private / unpublished skills | ✅ Point-to-point | ❌ Needs API access | ❌ Public only |
| Encryption | ✅ AES-256 | Depends | ❌ Plain |
| Dependencies | None (`openssl`, `curl`, `tar`) | SDK + API server | Package manager + net |

## License

MIT. See [LICENSE](LICENSE).

---

# C2C 协议 v2.0 — 中文

**智能体之间通过人类中转传授技能。**
**适用于任何能执行 shell 命令的智能体：Claude Code、Codex、OpenClaw 等。**

> 👤 **人类用户？** 请看 [§ 人类用户指南](#人类用户指南)。
> 🤖 **AI 智能体？** 直接跳到 [§ 安装（任意智能体）](#安装任意智能体)。

---

## 人类用户指南

> 🤖 **AI 智能体读者注意**：本章节是给人类操作者看的。请跳到 [§ 安装（任意智能体）](#安装任意智能体) — 那里才是你需要的可执行引导。

### 场景

你有多个 AI 编码智能体 — 一台机器上的 Claude Code、另一台上的 Codex、VPS 上的 OpenClaw 等等。你通过聊天（Telegram、微信、终端，或其他方式）分别和它们对话。其中一个智能体掌握着你想让另一个也学会的技能。C2C 能让你完成这个传授——**无需**对接任何 API，**无需**把技能发布到任何公开仓库，**你也无需**亲自登录任何机器。

### 你要做的 — 4 步，不碰任何 shell 命令

1. **让两个智能体学会如何传授技能。** 分别对每个智能体说：
   > *"去 https://c2cprotocol.org 按 Install 小节安装 lobster-distill 技能。"*

   智能体会读取本页面，按照 Install 小节配置好自己。每个智能体做一次即可。

2. **让发送方发起传授。** 对发送方智能体说：
   > *"用 lobster-distill 把我的 `<技能名>` 技能分享给另一个智能体。"*

   智能体会打包、加密、上传，然后给你回一条消息。消息里有两个清晰分隔的部分：`═══ 管理员请读 ═══`（给你的摘要）和 `═══ 目标龙虾请读 ═══`（接收方会执行的完整指令）。

3. **你来中转这条消息。** 在 Telegram / 微信 / Signal / Discord 里长按 → 转发，或者复制粘贴到邮件、短信，甚至电话念给对方 — 任意能发文字的通道都行。两个智能体**不需要**在同一平台上。

4. **接收方自动安装技能。** 接收方智能体看到 `目标龙虾请读` 那段，执行其中的命令，回复"已安装"。完成。

### 注意事项

- 下载链接**10 分钟 – 24 小时**后过期，请尽快转发。
- 消息里含有解密密码。**密码 + 链接** 是一对，缺一不可，都要送到接收方手里。
- 在 `管理员请读` 摘要里你能看到即将被分享出去的完整内容。**如果发现不对，就别转发**。
- 只有你主动转发，传授才会发生。你就是最后的把关者。

### 为什么选 C2C 而不是 …

- … 自己登录每台机器？你根本不需要碰 shell。
- … 把技能发布到公开仓库？技能不会公开，纯点对点。
- … 基于 API 的协议（Google A2A 等）？智能体之间从不直接通信，你始终掌控全程。

---

## C2C 是什么？

C2C 是一个协议，用于把一个技能（一个代码+文档的目录）从一个 AI 编码智能体传授到另一个，由人类在任意 IM 平台上转发一段简短的 Notes 完成。

- **智能体之间没有直接网络连接**：发送方从不与接收方的机器通信
- **人类即路由器**：管理员复制粘贴 Notes — 这就是完整的传输通道
- **AES-256 加密**：技能包已加密，只有拿到 Notes 中密码的接收方智能体能解密
- **自动过期**：加密包在 10 分钟 – 24 小时内自动删除

## 参考实现：lobster-distill

官方实现位于 **https://github.com/qiaoy01/lobster-distill**。本页包含安装和使用所需的完整说明 — 无需跨页查看。

## 安装（任意智能体）

直接从 qiaoy01 仓库获取两个 shell 脚本：

```bash
mkdir -p skills/lobster-distill
cd skills/lobster-distill
curl -fsSL -O https://raw.githubusercontent.com/qiaoy01/lobster-distill/main/share.sh
curl -fsSL -O https://raw.githubusercontent.com/qiaoy01/lobster-distill/main/receive.sh
curl -fsSL -O https://raw.githubusercontent.com/qiaoy01/lobster-distill/main/SKILL.md
chmod +x share.sh receive.sh
```

依赖：`openssl`、`curl`、`tar`（系统自带，无需安装）。

## 发送技能

在智能体工作目录下运行：

```bash
bash skills/lobster-distill/share.sh <技能目录或文件> "简要描述"
```

`share.sh` 会把目标打包为 tar.gz，用 AES-256-CBC + PBKDF2 加密，上传到临时存储（优先 `c2cprotocol.org/share`，10 分钟过期；失败 fallback 到 `litterbox.catbox.moe`，24 小时），并输出带两个清晰分隔块的 Notes：

- `═══ 管理员请读 ═══` — 人类看的摘要
- `═══ 目标龙虾请读 ═══` — 可直接运行的 `curl`+`openssl`+`tar` 命令

管理员将整段 Notes 原样转发（长按→转发、复制粘贴，甚至电话念给对方）给目标智能体，任意文本通道皆可：Telegram、微信、Discord、Signal、邮件、短信等。

## 接收技能

目标智能体可以直接运行 Notes 中的命令（标准 `curl`/`openssl`/`tar`），或使用辅助脚本：

```bash
bash skills/lobster-distill/receive.sh <URL> <密码> <技能名> tar
```

技能会安装到 `skills/<技能名>/`。

## 工作流程

```
发送方              人类管理员             接收方
  │                    │                    │
  │ 1. share.sh        │                    │
  │ 2. 输出 Notes ───→ │                    │
  │                    │ 3. 转发 Notes ───→ │
  │                    │   （任意 IM/邮件）  │
  │                    │                    │ 4. receive.sh
  │                    │                    │ 5. 技能已安装 ✅
```

## 各智能体安装路径

所有智能体使用同一套脚本，区别只在 `skills/` 目录约定：

### Claude Code
- 用户级：`~/.claude/skills/<name>/`
- 项目级：`.claude/skills/<name>/`
- 在任意 shell 会话运行 `share.sh` / `receive.sh`，Claude Code 在下次会话自动加载新技能。

### Codex
- 将技能放在项目根的 `skills/<name>/`，在 prompt 或配置中引用。

### OpenClaw
- 技能位于智能体工作目录的 `skills/<name>/`。`share.sh` 和 `receive.sh` 默认就是这个布局。

### 其他智能体
- 只要能运行 shell 命令就能用 C2C。把解压命令（`tar xzf ... -C <目录>`）的目标指向你的智能体读取技能的目录即可。

## 安全说明

- **加密**：AES-256-CBC + PBKDF2 + 随机 salt。24 字符 base64 一次性密码，嵌入 Notes。
- **人类参与审核**：没有管理员主动转发，什么都不会发。管理员在转发前能看到完整内容。
- **无智能体直连**：与基于 API 的协议（如 Google A2A）不同，发送方与接收方之间没有直接网络连接。
- **命令可见**：Notes 中是纯粹的 `curl` / `openssl` / `tar` 调用，接收方智能体（或人类）可以在执行前逐步审查。
- **临时文件清理**：`share.sh` 与 `receive.sh` 只清理自己在 `/tmp/` 下的文件。

## 对比

| 特性 | C2C (lobster-distill) | Google A2A | 包管理器 |
|---|---|---|---|
| 跨 IM 平台 | ✅ 任意文本通道 | ❌ 仅 API | ✅ 仅 Web |
| 人类审核 | ✅ 必须 | ❌ AI 直连 | ❌ 自动安装 |
| 私有/未发布技能 | ✅ 点对点 | ❌ 需 API 接入 | ❌ 仅公开 |
| 加密 | ✅ AES-256 | 取决于实现 | ❌ 明文 |
| 依赖 | 无（`openssl`、`curl`、`tar`） | SDK + API 服务 | 包管理器 + 网络 |

## 许可证

MIT。见 [LICENSE](LICENSE)。
