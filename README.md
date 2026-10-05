# codex-remote

## 快速开始

### 1. 安装依赖

```bash
pnpm install
```

### 2. 生成配置

源码开发模式下：

```bash
pnpm run build
pnpm start -- init
```

如果以后发布为 npm 包，则可以使用：

```bash
npx codex-remote init
```

### 3. 填写 QQ Bot 凭据

编辑 `.env`：

```env
QQBOT_APP_ID=你的AppID
QQBOT_CLIENT_SECRET=你的ClientSecret
QQ_CODEX_ALLOWED_C2C_SENDERS=你的QQ用户OpenID
QQ_CODEX_PERMISSION_ADMIN_SENDERS=你的QQ用户OpenID
```

QQ Bot 可以在 [QQ 开放平台](https://q.qq.com/qqbot/openclaw/index.html) 创建并获取 AppID / AppSecret。
默认访问控制为 `deny-by-default`；如果不配置 allowlist，bridge 会启动，但不会接受聊天侧任务。

### 4. 构建并启动

```bash
pnpm run build
pnpm start
```

常用运行时命令：

```bash
pnpm start -- status
pnpm start -- doctor
pnpm start -- logs 200
pnpm start -- tasks 20
pnpm start -- task <taskId>
pnpm start -- deliveries 20
pnpm start -- stop
pnpm start -- restart
```

默认 turn 硬超时为 30 分钟，工具连续 5 分钟无事件会被中断。可通过
`QQ_CODEX_TURN_TIMEOUT_MS` 和 `CODEX_TOOL_SILENCE_TIMEOUT_MS` 调整。

本地 smoke 测试时可以禁用 QQ gateway：

```powershell
$env:QQ_CODEX_DISABLE_QQ_GATEWAY='1'
pnpm start
```

## 配置示例
### 项目别名

```env
QQ_CODEX_PROJECT_ALIASES_JSON={"codex-remote":{"cwd":"D:/Project/github/codex-remote","label":"Codex Remote"}}
```

配置后可在 QQ 中使用：

```text
/aliases
/new codex-remote 修复当前 TypeScript 类型错误
```

### 访问控制

默认使用 `deny-by-default`。至少配置一个允许的私聊发送者、群或群成员：

```env
QQ_CODEX_ALLOWED_C2C_SENDERS=OPENID1,OPENID2
QQ_CODEX_PERMISSION_ADMIN_SENDERS=OPENID1
QQ_CODEX_ALLOWED_GROUPS=GROUP_OPENID1
QQ_CODEX_GROUP_REQUIRE_MENTION=true
```

也可以显式指定：

```env
QQ_CODEX_ACCESS_CONTROL=deny-by-default
```

只有明确设置 `QQ_CODEX_ACCESS_CONTROL=allow-all` 才会放开全部来源；`doctor` 会对此给出安全警告。


## 架构概览

```text
       Q Q 
        |
        v
Bridge daemon
        |
        +-- Command Router
        +-- Session Store / Transcript Store / Runtime State
        +-- Access Control
        |
        v
Codex app-server driver
        |
        v
Codex Desktop threads and tool calls
```
当前实现已经把 Codex 回合纳入任务状态、session/thread 调度、工具事件、取消、超时、重试、投递重试和重启恢复链路。

## 开发

```bash
git clone <你的仓库地址>
cd codex-remote
pnpm install
cp .env.example .env
pnpm run build
pnpm start
```

常用检查：

```bash
pnpm run check
pnpm test
pnpm run test:offline
pnpm run test:bridge-smoke
```

## 安全提醒

- `.env`  QQ Bot不要提交到仓库。
- 本项目会处理聊天消息、附件、语音和本地文件路径，联调时注意隐私边界。
- 如果把仓库公开，先检查历史提交中是否出现过真实 token 或本地路径。

## 来源与致谢

本项目的起点是社区开源项目 [`qq-codex-bridge`](https://github.com/983033995/qq-codex-bridge)，(https://github.com/983033995)。QQ 机器人与 Codex Desktop 之间的桥接思路，以及消息收发、媒体处理、会话与线程管理等基础骨架，都由该项目最早建立。

codex-remote 在此基础上重新定位为面向多个聊天入口的本地调度层，新增并重构了 Codex app-server 链路、Turn Manager 状态机、投递重试与恢复、微信文本网关等能力；同时完整保留上游的 MIT 许可与版权声明。

感谢 Codex Desktop 与 QQ 开放平台提供的能力支持，也感谢上游项目及其所有贡献者的工作。

## License

本项目使用 [MIT License](./LICENSE)。
