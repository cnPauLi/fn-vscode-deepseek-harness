# DeepSeek Harness for VS Code

一个零依赖的 VS Code 扩展，把 **DeepSeek Harness (DSH)** 接入 VS Code 的两种形态：

1. **忠实窗口**：把 DSH 的 Web GUI 原样内嵌到 VS Code 侧边栏 / 辅助侧边栏 / 编辑器标签页，接入**你已经启动好的** DSH 服务（扩展不启动、不安装、不重启 dsh）——不注入脚本、不改写界面、不拦截交互，不影响你对 DSH 的页面组织、第三方插件装配等任何二次开发行为；
2. **Copilot 桥接（v0.7.13 起，早期版本）**：把 DSH 注册为 VS Code 聊天模型——模型选择器里出现 **DSH (DeepSeek Harness)、DeepSeek-V4-Pro (DSH)、DeepSeek-V4-Flash (DSH)、deepseek-v4-flash-vision-exp (DSH)** 等条目，选中即可在 Copilot Chat 里借助 DSH 强大的任务编排与工具调用能力解题。

> **Copilot 桥接不影响「忠实窗口」形态**——它只是为便捷编程而做的功能提升；你不选这些模型条目时，一切与没有桥接功能时完全一样。

如果喜欢本扩展请转至 [Deepseek-Harness-for-VS-Code](https://github.com/Vithrive/Deepseek-Harness-for-VS-Code) 星标助力；对 Chrome Extension 有需求也请关注 [Deepseek-Harness-for-Chrome](https://github.com/Vithrive/Deepseek-Harness-for-Chrome)。

> **版本适配**：本扩展适配 **dsh v0.1.2-rc.1 及以上版本**——自动完成该版本起新增的 Web 浏览器认证（扩展受管认证代理，面板与 Copilot 桥接免手动登录，详见下文「dsh web 浏览器认证」）；同时**向下兼容**未启用认证的旧版 dsh（RPC 端点新旧格式自动回退）。

## 🙏 致谢

- [Pelapis](https://github.com/Pelapis)——贡献 macOS 面板剪贴板快捷键修复并迭代收敛作用域（内置插件 `dsh-webview-clipboard`，PR #11、#14）。
- [curtainsmall](https://github.com/curtainsmall)——修复面板 iframe 非整数倍缩放的整页模糊（改用 CSS zoom，PR #10）。
- [anupamme](https://github.com/anupamme)——报告工作区设置注入面，推动子进程调用安全加固（PR #12）。

---

## 🚀 快速安装

- **.vsix**：从 [GitHub Releases](https://github.com/cnPauLi/fn-vscode-deepseek-harness/releases/latest) 下载 `fn-vscode-deepseek-harness-<版本>.vsix`，然后：

  ```bash
  code --install-extension fn-vscode-deepseek-harness-<版本>.vsix
  ```

  或在 VS Code 中：`Ctrl+Shift+P` → `Extensions: Install from VSIX...`。

安装后 `Ctrl+Shift+P` → `Reload Window`。然后**自行启动 DSH 服务**（例如 `dsh --profile web --port 8080`），并把 `dshPanel.url` 指向它（默认 `http://127.0.0.1:3080`）——本扩展只负责接入，不会替你启动 dsh。

---

## 🪟 忠实窗口（面板）

- 把 DSH Web GUI 原样内嵌到侧边栏 / 辅助侧边栏 / **编辑器标签页**（标签页可 Pin 住；与侧边栏「单活动视图」自动让位，规避 DSH 前端 webview 单实例限制）；
- **只接入已启动的 DSH**：`dshPanel.url` 上确认有服务在听才渲染，避免白屏；服务没起或地址不对时给出明确提示。扩展**不启动、不安装、不重启** dsh，也不要求本机存在 `dsh` 命令行；
- **工作区自动对接**：把 VS Code 当前工作区注册到 DSH 工作区列表（幂等，不覆盖你在 DSH 里的手动选择）；
- **远程支持**：Remote-SSH / Dev Containers 下运行于服务器端，经端口转发把服务器上已启动的 DSH 接入本地 VS Code；
- 面板按钮：刷新（不打断运行中的任务）/ 在浏览器中打开；字号跟随 `editor.fontSize` 等比缩放（CSS zoom 实现，非整数倍缩放同样清晰）；
- **发送选中内容 / 拖放文件到 DSH 对话框**（自动安装配套插件 `dsh-drop-caret`）：把文件、文件夹、代码段以 `路径:行号` 引用精确插入对话框光标处——**从 VS Code 资源管理器拖拽直接引用源文件本身**（不产生副本）；从系统文件管理器拖入时浏览器无法取得真实路径，此时才回退为工作区 `.dsh-drop/` 下的内容快照。点击 DSH 对话中的外链在系统浏览器打开（配合 DSH 插件 `dsh-open-links`）。
- **macOS 剪贴板快捷键修复（自动安装配套插件 `dsh-webview-clipboard`）**：修复 macOS 上面板内 ⌘C/⌘V/⌘X 失效的问题——DSH 页面以跨源 iframe 内嵌于 webview 时，浏览器的原生剪贴板默认动作不会发生。插件注入 DSH 页面后拦截这三个键并经 execCommand 显式执行。仅 macOS + 被内嵌时启用，其余环境行为不变。

### 使用示例：发送选中内容到对话框

拖拽 / 右键发送是 `dsh-drop-caret` 最常用的能力，操作如下：

1. 在 VS Code 中**框选住代码块 / 文字块**；
2. **右键**，点击 **「DeepSeek Harness: 发送选中内容到对话框」**：

   ![右键菜单：发送选中内容到对话框](media/send-selection-menu.png)
3. 代码块所在行数的链接（`路径:起始行-结束行`）就会被发送到对话框，插入在当前光标位置：

   ![发送结果出现在 DSH 对话框中](media/send-selection-result.png)
4. 在 DSH 里直接发送消息即可，模型可通过引用精确定位到代码块所在文件与行号。

> 同样地，也可以把文件 / 文件夹从系统文件管理器或 VS Code 资源管理器**直接拖进**对话框，插入位置同样是拖放点对应的光标位置。

### 面板相关配置

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| `dshPanel.url` | `http://127.0.0.1:3080` | **要接入的 DSH 服务地址（扩展侧访问用）**。扩展不会自动启动 dsh，需你自行先启动服务，如 `http://127.0.0.1:8080`；工作区注册与 Copilot 桥接走这里 |
| `dshPanel.externalUrl` | 空 | **浏览器可达的 DSH 代理地址**，如 `https://thinknas:5667/app/lhp-code-server/proxy/3080/`。填写后面板与标签页的 iframe 直接加载它，不再使用受管认证代理——浏览器版 VS Code / Remote 下回环地址在 webview 侧不可达，此时就填这个 |
| `dshPanel.authTokenFile` | 空 | DSH 的启动凭据：可填**令牌文件路径**，也可**直接填令牌本身**或含 `token=` 的认证链接。留空=不启用 |
| `dshPanel.autoRegisterWorkspace` | `true` | 是否把当前工作区自动注册为 DSH 工作区 |
| `dshPanel.installClipboardPlugin` | `true` | 是否写入内置 `dsh-webview-clipboard` 插件（修复 macOS 面板内编辑快捷键；Windows/Linux 上为惰性文件不影响行为）；是否启用由 DSH 侧决定 |
| `dshPanel.dshHome` | 空 | DSH 的 home 目录（其下有 `profiles/web`），配套插件（`dsh-drop-caret` / 剪贴板兼容插件）写到这里。留空=取 `DSH_HOME` 环境变量，再回退 `~/.dsh`；DSH 由别的程序托管（如 fnOS 打包应用）时默认位置不存在，扩展会**跳过插件安装且不报错**，要把插件装到该 DSH 上就把它真实的 home 填进来 |

> 本扩展已移除全部「主动启动 dsh」相关能力与配置（`autoStart` / `autoInstallDsh` / `dshCommand` / `killOnDispose` / `openSystemBrowser` / `host` / `port` 及「重启 dsh web」按钮）：服务生命周期完全由你掌控。

### dsh web 浏览器认证（扩展自动完成，无需手动登录）

dsh `0.1.2-rc` 起为 Web GUI 启用了浏览器认证：每次 `dsh web` 启动会生成一个一次性「进程启动令牌」并打印形如 `dsh web: http://127.0.0.1:3080/?token=…` 的认证链接，浏览器打开该链接后换取签名 Cookie，此后凭 Cookie 访问；裸地址一律返回 401。同时 `/api` 还有浏览器信任围栏（Host 必须回环、Origin 与 Host 一致、拒绝跨站请求）。

扩展的处理方式（**不关闭 dsh 的任何安全机制**）：

- 在本机 `127.0.0.1` 随机端口启动一个**受管认证代理**：由代理完成令牌 → Cookie 换发，之后给每个转发请求（页面、API、WebSocket）注入凭据，面板与 Copilot 桥接全部改走代理；
- **令牌来源**（按优先级）：`dshPanel.authTokenFile`（填文件路径时，dsh 重启换令牌可自动跟随；直接填令牌或含 `token=` 的认证链接同样可用）→ 上次会话在 VS Code 全局状态里的缓存 → 面板提示时手动粘贴认证链接；
- 因为扩展不再启动 dsh，**读不到它的 stdout**。若 dsh 由外部进程托管（例如 fnOS 打包应用把令牌写到 `var/gateway/web.token`），把该文件路径填进 `dshPanel.authTokenFile` 即可自动跟随、免手动粘贴；
- 拿不到令牌时，面板提示一次（3 分钟冷却）：可点「粘贴认证链接」，或把令牌/令牌文件路径填进 `dshPanel.authTokenFile`；
- 「在浏览器中打开」按钮会自动携带当前令牌，系统浏览器可正常换取自己的 Cookie；
- Remote / 非回环地址场景不启用代理（认证须在 dsh 所在机器的浏览器完成一次）。

### 对旧版本 dsh 的兼容（无认证版本）

扩展对未启用 web 认证的旧版 dsh 保持兼容，回退路径自动：

- **认证链路**：面板加载前会探测首页状态——旧版返回 200（无认证）即走原直连路径，不启用代理注入；认证引导也只在探测到 401 时出现；
- **RPC 端点**：扩展按新版斜杠端点（`workspace/create` 等）请求，收到 404 自动回退旧点号端点（`workspace.create`）；`session/page` 不可用时回退 `session.history`。

### 远程服务器（vscode-server）场景

扩展声明 `extensionKind: ["workspace"]`，在 Remote-SSH / Dev Containers 等场景下运行于服务器端：

1. **服务需在服务器端已经启动**（扩展不会替你安装或启动 dsh）；
2. 自动端口转发：通过 `vscode.env.asExternalUri` 把远程 `127.0.0.1:3080` 暴露到本地，iframe 直接加载，无需手动配 SSH 隧道（首次转发确认允许即可）；
3. 把 `dshPanel.url` 指向服务器上的 DSH 地址；当前工作区会自动注册进 DSH 工作区列表。

如果 DSH 跑在另一台机器、且不是通过 VS Code Remote 连接的，可手动建隧道：`ssh -L 3080:127.0.0.1:3080 user@server`。

### 浏览器版 VS Code / Remote 下面板打不开？配 `dshPanel.externalUrl`

面板是把 DSH 页面放进 iframe 加载的，而这个 iframe 由 **webview 侧**（浏览器或你本机）去访问。本地桌面场景 webview 与扩展宿主同机，`http://127.0.0.1:3080` 就能开；但在**浏览器版 VS Code（如 `https://thinknas:5667/app/lhp-code-server/`）或 Remote-SSH** 下，webview 在客户端，回环地址指向的是客户端自己的本机——通常什么都没有，表现为面板与标签页**一片空白**。

此时把「浏览器里能打开 DSH 的那个地址」填进 `dshPanel.externalUrl` 即可，例如：

```jsonc
{
  "dshPanel.url": "http://127.0.0.1:3080",                                  // 扩展侧访问（工作区注册 / Copilot 桥接）
  "dshPanel.externalUrl": "https://thinknas:5667/app/lhp-code-server/proxy/3080/"  // webview 侧加载
}
```

填写后面板与标签页的 iframe **直接加载该地址**，不再使用受管认证代理（它的回环地址在客户端本就不可达）。认证由这条代理链路在浏览器侧完成——先用面板顶部的「在浏览器中打开」访问一次、让浏览器拿到 DSH 的 Cookie，面板内即可正常加载。

> 前提：该地址本身在浏览器里能打开（例如证书受信任）。若浏览器控制台报
> `Could not register service worker ... SSL certificate error`，那是 VS Code Web 的 webview 依赖 Service Worker、而站点证书不受信任所致，与本扩展无关（[code-server#3410](https://github.com/coder/code-server/issues/3410)）。需要让证书受信任，或改用 Firefox。

---

## 🧭 Copilot 桥接：操作指南

### 快速上手

1. 打开 Chat 面板（`Ctrl+Alt+I`）→ 模型选择器（`Ctrl+Alt+.`）里选择 **DSH (DeepSeek Harness)**（或直接选 **DeepSeek-V4-Pro (DSH)** 等固定条目）；
2. 直接提问，例如「帮我分析这个项目的数据」——DSH 用其配置的模型在工作区执行任务、调用工具解题，答案**流式回写**聊天框；
3. 每个 Copilot 聊天对应一个 DSH 会话：**新聊天自动新建 DSH 会话，同一聊天内持续追问复用同一会话**；你可以在 DSH 面板里实时看到完整执行过程。

### 模型与推理档位

- **模型**：`DSH (DeepSeek Harness)` 条目默认跟随 DSH 设置里的默认模型（`agent-default-model`）；也可用 `dshPanel.chatProvider` / `dshPanel.chatModel` 指定（如 `deepseek-official` / `deepseek-v4-pro`，需先在 DSH 设置中配置好对应 provider）。模型选择器里的 **DeepSeek-V4-Pro (DSH)** 等条目则固定对应 DeepSeek 官方模型。
- **推理档位（reasoningEffort）**：在聊天界面的模型配置里选择（off / low / high / max，与 DSH 会话同步生效）；`dshPanel.dshReasoningEffort` 作为兜底配置。

### 切换模型再切回

Copilot 会话中途切到其他自定义模型问答、再切回 DSH 模型时，扩展会把「其他模型产出的中间对话」**打上产地标签补发给 DSH 会话**；DSH 自己答过的内容不会重复回传（省 token、不占上下文）——DSH 侧时间线保持完整。

### 常用命令

| 命令 | 作用 |
| --- | --- |
| `DeepSeek Harness: 重置 DSH 会话映射` | 清空「聊天 → DSH 会话」映射，下次提问创建全新 DSH 会话 |
| `DeepSeek Harness: 检查 DSH 状态` | 查看 DSH 是否可达、模型提供方是否注册、当前模型配置 |
| `DeepSeek Harness: 诊断 DSH 模型注册表` | 导出模型注册表诊断数据（排查用） |

> 取消等待不会杀掉 DSH 任务：任务会继续在 DSH 中运行，可到面板查看。

### 桥接相关配置

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| `dshPanel.enableDshModel` | `true` | 是否注册 DSH 聊天模型条目（关闭则桥接不生效，面板不受影响） |
| `dshPanel.chatProvider` / `dshPanel.chatModel` | 空 | `DSH (DeepSeek Harness)` 条目使用的 provider / 模型（如 `deepseek-official` / `deepseek-v4-pro`）；留空跟随 DSH 默认 |
| `dshPanel.chatAgentPreset` | 空 | DSH 会话创建时使用的 agent 预设（如 `liangshen`）；留空=DSH 默认 |
| `dshPanel.dshReasoningEffort` | 空 | 推理档位兜底：off / low / high / max；界面选择优先 |
| `dshPanel.chatTimeoutMs` | `900000` | 单次任务最长等待毫秒数（15 分钟），超时后任务仍在 DSH 面板运行 |
| `dshPanel.chatSyncLookbackMin` | `60` | 聊天会话文件扫描窗口（分钟） |
| `dshPanel.debugModelMessages` | `false` | 调试：把 VS Code 发给模型的消息结构写入 `.dsh-debug/` |

---

## 🧩 Copilot 桥接：实现原理

整体数据流：

```
Copilot Chat（VS Code 组织好的对话）
        │  语言模型提供方协议（vscode.lm.registerLanguageModelChatProvider）
        ▼
本扩展（dsh 提供方）
  1. 滤除杂音：剥离系统提示词、工具定义、环境/上下文包裹（<prompt>/<userRequest>/<instructions>…），
     只保留真实问答与 Copilot 记忆正文
  2. 会话映射：以 Copilot 聊天的 sessionId 为键，映射到 DSH 会话（一聊天一会话）
  3. 增量同步：只把 DSH 尚未见过的内容发给 DSH（自己答过的不回传；其他模型的问答打产地标签补发）
  4. 档位同步：把界面选择的 reasoningEffort 传给 DSH（session.selectModel）
        │  session.create / session.prompt / session.history（DSH RPC）
        ▼
DSH：用自己的一套 harness（记忆 / 技能 / AGENTS.md / 工具 / agent 预设）二次组织，交给配置的模型执行
        │  流式事件（text-delta）
        ▼
本扩展：增量流式回写 Copilot 聊天框
```

要点：

- **滤除杂音**：VS Code 交给模型的每条消息可能包裹 `<instructions>`（.copilot/instructions、AGENTS.md 引用）、`<prompt>` 真实提问、`<userMemory>/<sessionMemory>` 记忆块等。扩展只提取真实提问与记忆正文——上下文组织交给 DSH 自己的 harness，避免两套 harness 互相干扰。
- **会话映射（sessionId 直接映射）**：Copilot 每个聊天在磁盘上有唯一文件 `workspaceStorage/<哈希>/chatSessions/<sessionId>.jsonl`（文件名即 sessionId）。扩展以 `m-<sessionId>` 为键建立「聊天 → DSH 会话」的一对一映射：
  - 非首轮：用「文件最后一条提问 == 当前转录的上一轮提问」认领聊天文件（上一轮必然已落盘，零竞态、零等待）；
  - 首轮：新聊天文件此刻只有元数据，直接认定「最近 60 秒内新建的空聊天文件」为当前聊天；
  - 兼容 Windows / macOS / Linux，以及 vscode-server（Remote-SSH / WSL / Dev Containers）等不同用户数据目录，并优先匹配当前工作区；
  - 兜底：请求落盘竞态等极少数情况退回首问哈希，并配合转录校验防串线。
- **增量同步（省 token）**：DSH 会话自己会回放已答内容，因此扩展只发送「最后一条 DSH 答案之后的新增内容」——连续对话时只发新提问；切走再切回时，外来问答以 `【Copilot 其他模型回答】` 标签补发。
- **双投递去重**：VS Code 会把同一次提问投递两次（裸提问 + 带上下文），扩展识别为同一问题后只执行一次，另一路直接回放同一份答案。
- **并发支持**：多个聊天同时使用 DSH 模型时，各聊天独立定位、独立会话、并行返回；扩展对启动探测、文件解析做了记忆化与缓存，避免并发互相拖慢。

---

## 🌱 版本状态声明

Copilot 桥接是**早期版本**，但已经过充分测试、**功能完全可用**：

- 欢迎大家在不同操作系统（Windows / macOS / Linux，以及 Remote-SSH、WSL、Dev Containers 等远程场景）中测试使用；
- 如遇问题请在 [GitHub Issues](https://github.com/Vithrive/Deepseek-Harness-for-VS-Code/issues) 提出，作者会尽快回复和改进；
- 再次强调：**Copilot 桥接不影响「忠实窗口」形态**——面板始终忠实呈现 DSH Web GUI，不对页面注入、改写或拦截任何东西，也不干涉你对 DSH 的插件开发与界面定制。

---

## 🔧 从源码安装（开发模式）

本扩展是纯 JavaScript，不需要 npm install、不需要编译：

```bash
git clone https://github.com/cnPauLi/fn-vscode-deepseek-harness.git
code fn-vscode-deepseek-harness
```

在 VS Code 中按 `F5` 打开扩展开发宿主窗口，在其中打开你的项目文件夹即可。自行打包安装：

```bash
npx --yes @vscode/vsce package --allow-missing-repository
code --install-extension fn-vscode-deepseek-harness-<版本>.vsix
```

## 前置条件与已知限制

- **前置条件**：目标机器上**已经有一个在运行的 DSH 服务**（例如 `dsh --profile web --port 8080`，或由 fnOS 打包应用等外部进程托管），并把 `dshPanel.url` 指向它。本扩展既不要求本机存在 `dsh` 命令行，也不会替你安装或启动 dsh。另外 DSH 默认响应头未设置 `X-Frame-Options` / 严格 CSP，可被 iframe 正常内嵌。
- **已知限制**：DSH 前端在 VS Code webview 多实例下退化为单例（普通浏览器多开正常，属 DSH 前端实现层面问题），因此标签页与侧边栏暂不能同时加载 DSH；扩展以「单活动视图」策略规避（打开标签页时侧边栏自动让位显示占位，关闭后自动恢复）。
