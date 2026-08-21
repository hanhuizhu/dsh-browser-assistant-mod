# dsh-browser-assistant-mod

dsh 浏览器助手的 Chrome 扩展魔改版。相对原版：**自动批准操作（无确认弹窗）、自动跟随当前标签页**。

魔改版仓库：`https://github.com/hanhuizhu/dsh-browser-assistant-mod`

## 一、工作原理

一句话：**LLM 的指令从 dsh 发出，经 bridge 的 WebSocket 送到扩展，扩展驱动浏览器页面变化；页面变化后扩展把结果经同一条 WebSocket 送回 dsh，LLM 再决定下一步。**

完整消息流：

```
① LLM 决定操作         ② dsh 桥插件收到指令         ③ WebSocket 发送指令
   browser_click/         打包成消息，走               (ws://127.0.0.1:PORT/
   browser_type/           /ext/bridge 通道             ext/bridge, token 认证)
   browser_navigate …     (token 认证)
        │                       │                             │
        ▼                       ▼                             ▼
                                                         ④ Chrome 扩展执行
                                                            在你当前标签页上
                                                            点击/输入/导航/滚动
                                                                    │
                                                                    ▼
                                                         ⑤ 浏览器页面变化
                                                                │
        ┌───────────────────────────────────────────────────────┘
        ▼
   ⑥ 扩展读取变化后的页面 → 渲染成结构化文本快照
   ⑦ 经同一条 WebSocket 送回 dsh 桥插件
   ⑧ dsh 返回给 LLM → 作为下一步依据
```

要点：

- **LLM 不截图、只读文本**。页面被渲染成**结构化文本快照**——标题、正文、带编号的可交互清单（`[1]` `[2]`…）、表单字段（敏感值打码）。这就是 LLM"看到"的"页面"。
- **bridge 是中间人**。它一头连 dsh 的 LLM/工具层，一头连 Chrome 扩展，全靠一条 **token 认证的 WebSocket（`/ext/bridge`）** 中转：LLM 的指令走这条通道下去，扩展的结果也走这条通道上来。
- **LLM 命令 → 页面变化**：LLM 调 `browser_*` → 桥插件 → WebSocket → 扩展在你真实标签页执行 → 页面真正改变（登录态保留）。
- **页面变化 → 消息回 LLM**：扩展读取变化后的页面（或操作结果），经同一 WebSocket 送回桥插件 → dsh 返回给 LLM。
- **魔改点**：`background.js` 里 `Fe()` 恒返回"批准"（跳过确认弹窗）；`observeActive()` 切标签页时自动跟随新页面（不再暂停询问）。

## 二、安装方式

> 整套由两部分组成：**dsh 浏览器桥插件**（挂在 dsh 宿主上，提供 `browser_*` 工具和 WebSocket 通道）+ **Chrome 扩展**（魔改版）。两者缺一不可。

### 环境依赖
- Chrome ≥ 116、Node.js（LTS）+ pnpm、rsync / curl / tar
- 本机已能运行 dsh（dsh Desktop 或 `npx @deepseek-ai/dsh web`）

### 第 1 步：装 dsh 浏览器桥插件
```bash
git clone https://github.com/Lum1104/dsh-browser.git && cd dsh-browser && ./scripts/install.sh
```
脚本会构建插件、注册进 dsh 的 `web` profile、复制原版扩展到 `~/.dsh/browser-extension` 并打开 `chrome://extensions`。

### 第 2 步：启动 dsh 宿主
```bash
cd ~/.dsh/dsh-browser && pnpm start     # 固定版本
npx @deepseek-ai/dsh web                # 或最新公开版
```

### 第 3 步：加载本魔改版扩展
拉取魔改版仓库，把文件覆盖到宿主已加载的扩展目录，然后在 `chrome://extensions` 找到"dsh Browser Assistant"点**重新加载**：

```bash
git clone https://github.com/hanhuizhu/dsh-browser-assistant-mod.git
cd dsh-browser-assistant-mod
rsync -a --delete-after ./ ~/.dsh/browser-extension/
```

> 之前加载过原版，请先移除再加载本版本，避免同名冲突。若不想覆盖到目录，也可以在 `chrome://extensions` → 开发者模式 → 加载已解压的扩展程序 → 直接选魔改版仓库目录。

### 第 4 步：使用
1. 打开扩展侧边栏（自动探测本机 dsh，回环免 token）。
2. 打开任意页面，让 dsh 读取快照，即可用 `browser_snapshot` / `browser_click` / `browser_type` / `browser_navigate` / `browser_scroll` 等工具操作该页。
3. 若 dsh 监听在非 3080/3081/3090 端口，在侧边栏设置里手动填桥地址。
