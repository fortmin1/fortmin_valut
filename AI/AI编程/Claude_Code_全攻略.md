# Claude Code 从 0 到 1 全攻略

> **视频来源：** 哔哩哔哩 - 马克的技术工作坊  
> **视频链接：** https://b23.tv/bfxfvnz  
> **整理时间：** 2026-04-21  
> **字幕获取方式：** yt-dlp 下载音频 + Whisper 语音识别

---

## 📋 目录

- [第一部分：环境搭建与基础交互](#第一部分环境搭建与基础交互)
- [第二部分：复杂任务处理与终端控制](#第二部分复杂任务处理与终端控制)
- [第三部分：多模态与上下文管理](#第三部分多模态与上下文管理)
- [第四部分：高级功能扩展与定制](#第四部分高级功能扩展与定制)

---

## 第一部分：环境搭建与基础交互

### 1.1 安装 Claude Code
- **官网：** anthropic.com/claude-code
- **安装：** 复制官网提供的安装命令，在终端执行

### 1.2 登录与授权
两种标准接入方式：
| 方式 | 说明 |
|------|------|
| **订阅制** | 购买 Claude Pro 或 Max 会员 |
| **API Key** | 按用量付费，用多少花多少 |

登录命令：`claude login`

> 💡 **提示：** Claude Code 本身不与 Claude 模型绑定，可以使用国产模型（GLM、MiniMax 等）驱动，只需设置几个环境变量。

### 1.3 三种交互模式

通过 `Shift+Tab` 循环切换：

| 模式 | 标识 | 行为 |
|------|------|------|
| **默认模式** | 显示 `? for shortcuts` | 每次创建/修改文件前都会询问用户 |
| **自动模式** | `Accept Additions` | 自动创建和修改文件，不询问，最方便 |
| **规划模式** | `Plan Mode` | 只讨论方案，不修改文件 |

### 1.4 授权选项（创建文件时）

| 选项 | 说明 |
|------|------|
| `Yes` | 单次授权，只同意创建当前文件 |
| `Yes, Allow All Additions this session` | 本次对话期间所有文件操作自动通过 |
| `No` | 不同意，可继续输入想法让 Claude 再生成 |

### 1.5 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Shift+Tab` | 切换模式（默认→自动→规划→默认...） |
| `Ctrl+G` | 打开 VSCode 编辑输入框 |
| `Shift+Enter` | 在输入框内换行（不提交） |
| `Ctrl+V` | 粘贴图片（注意：用 Ctrl+V，不是 Command+V，即使 Mac 也一样） |

---

## 第二部分：复杂任务处理与终端控制

### 2.1 在 Claude Code 内执行终端命令
- 直接输入命令即可，Claude Code 会进入 Bash 模式
- 可以执行 `open <文件>` 打开文件

### 2.2 规划模式 (Plan Mode) 详解
- 适用场景：大型重构、架构调整等需要先讨论方案的任务
- 使用 `Shift+Tab` 切换到 Plan Mode
- Claude 会先生成详细的执行计划，包含目标、清单、目录结构等
- 用户确认后才执行

### 2.3 跳过权限检测
```bash
claude --dangerously-skip-permissions
```
启动后模式会显示 `Bypass Permissions`，所有终端命令都不会再询问。

> ⚠️ **危险警告：** 此参数会让 Claude Code 彻底放飞自我，理论上拥有和你一样的终端权限，请谨慎使用。

### 2.4 后台任务管理

| 操作 | 命令/快捷键 |
|------|------------|
| 查看后台任务 | `Gun Tasks` 或 `Ctrl+C` |
| 关闭后台任务 | `K` |
| 启动后台任务 | `Ctrl+B` |

> 💡 **重要提示：** 开发服务器运行时会阻塞 Claude Code，需要按 `Ctrl+B` 将其放到后台

### 2.5 Rewind 版本回滚
- 命令：`/rewind` 或连续按 `Esc` 两次
- 会列出所有回滚点，选择后可以回滚代码和文件
- 只能回滚 Claude Code 自己写入的文件，终端命令生成的无法回滚
- 建议精准回滚使用 Git

---

## 第三部分：多模态与上下文管理

### 3.1 图片处理
两种方式添加图片：
1. **拖拽** - 直接把图片拖到 Claude Code 输入框
2. **粘贴** - `Ctrl+V` 粘贴图片（Mac 也用 Ctrl+V）

### 3.2 MCP (Model Context Protocol)
#### 安装 MCP Server
```bash
claude mcp install <server-name>
```
以 Figma 为例，安装后需要认证授权。

#### 使用 MCP 工具
- 查看已安装的 MCP：`/mcp`
- 使用 MCP 工具还原 Figma 设计稿，可以精确获取设计信息（组件样式、颜色、字体等）

### 3.3 恢复历史会话
| 命令 | 功能 |
|------|------|
| `/resume` | 恢复之前的对话 |
| `claude -C` | 启动 Claude Code 并自动恢复上一次对话 |

### 3.4 上下文压缩与清除

| 命令 | 功能 |
|------|------|
| `/compact` | 压缩上下文，保留重点信息 |
| `/clear` | 清空所有上下文内容 |

### 3.5 CLAUDE.md 项目记忆文件
- **作用：** 让 Claude Code 每次进入时自动读取项目信息和约定
- **生成：** `/generate-claude-md`
- **两种级别：**
  - **项目级别** - 放在项目目录，对当前项目生效
  - **用户级别** - 放在用户目录，对当前用户所有项目生效
- **查看/编辑：** `/memory`

---

## 第四部分：高级功能扩展与定制

### 4.1 Hook
**作用：** 在工具执行前/后/失败时执行自定义逻辑

#### 使用场景
- 代码格式化（如用 Prettier 自动格式化）
- 提交前检查等

#### 配置流程
1. `/hooks` 进入 Hook 配置
2. 选择执行时机（工具使用前/后/失败/发送通知等）
3. 选择要监听的目标工具（如 Write/Edit）
4. 填写具体的 Hook 命令
5. 选择保存范围：
   - **本地项目级别** - 放在 `.settings.local.js`，不随 Git 分发
   - **项目级别** - 放在 `.settings.js`，随 Git 分发
   - **用户级别** - 放在用户目录

#### Hook 传参示例
```bash
cat | jq '.filePath' | xargs prettier --write
```
- Claude Code 会传入 JSON，包含 `filePath` 等信息
- `jq` 解析 JSON 提取文件路径
- `xargs prettier --write` 格式化文件

### 4.2 Agent Skill
**本质：** 一个动态加载的 prompt，给大模型看的说明书

#### 创建方法
1. 在 `~/.claude/skills/` 下创建技能文件夹
2. 创建 `skill.md` 文件，包含：
   - `Name` - 技能名称
   - `Description` - 技能描述（Claude 根据此决定是否调用）
   - 具体指令内容

#### 使用方法
- **自动调用：** Claude 发现请求与 Skill 相关时自动调用
- **手动调用：** `/skill <name>` + 具体请求

#### 适用场景
与上下文关联大、对上下文影响小的任务（如写日报）

### 4.3 SubAgent
**本质：** 独立的 Agent，有自己独立的上下文、工具和 Skill

#### 创建方法
```bash
/make-agent
```
- 选择 Agent 类型（项目/用户级别）
- 选择创建方式（推荐用 Claude 初始化，或完全手动）
- 描述 Agent 要完成的任务
- 选择可用的工具（推荐 Read Only Tools）
- 选择模型和颜色标识

#### 适用场景
与上下文关联小、对上下文影响大的任务（如代码审核）

### 4.4 Agent Skill vs SubAgent 对比

| 特性 | Agent Skill | SubAgent |
|------|------------|----------|
| **上下文** | 共享主对话上下文 | 独立全新上下文 |
| **中间过程** | 记入主对话 | 不回传主对话 |
| **适用场景** | 写日报等 | 代码审核等 |
| **Token 消耗** | 可能塞满上下文 | 主对话保持干净 |

### 4.5 Plugin
**本质：** 打包一系列 Skill、SubAgent、Hook 的全家桶安装包

#### 使用方法
```bash
/clogging
```
- **Discover** - 发现新插件
- **Installed** - 查看已安装
- **Market Places** - 插件市场

#### 示例
**Friendly Design** 插件 - 一套前端 UI 设计规范，让生成的前端更符合现代审美

---

## 📌 常用命令速查表

### 基础命令
```bash
claude                    # 启动 Claude Code
claude login             # 登录
claude -C                # 启动并恢复上次对话
claude --plan            # 直接进入规划模式
claude --auto            # 直接进入自动模式
claude --dangerously-skip-permissions  # 跳过所有权限确认
```

### 会话管理
```bash
/resume                  # 恢复历史会话
/memory                  # 查看/编辑 CLAUDE.md
/generate-claude-md      # 生成 CLAUDE.md
/compact                 # 压缩上下文
/clear                   # 清空上下文
/rewind                  # 版本回滚
```

### MCP 相关
```bash
/mcp                     # 查看已安装的 MCP
claude mcp install <server>  # 安装 MCP 服务器
```

### Skill 与 Agent
```bash
/skills                  # 查看所有 Skill
/skill <name>            # 手动调用 Skill
/make-agent              # 创建 SubAgent
/agents                  # 查看所有 Agent
```

### Hook
```bash
/hooks                   # 配置 Hook
```

### Plugin
```bash
/clogging                # 插件管理器
```

### 后台任务
```bash
/gun-tasks              # 查看后台任务
Ctrl+B                   # 将当前任务放到后台
K                        # 关闭后台任务
```

### 其他
```bash
?                        # 显示快捷键帮助
```

---

## 🔧 工具安装（字幕识别流水线）

如果你想复用这个流程，需要安装以下工具：

```bash
# 1. yt-dlp - 下载视频音频
sudo apt-get install yt-dlp

# 2. Whisper - 语音识别
sudo pip3 install openai-whisper --break-system-packages

# 3. 下载 B 站视频音频
yt-dlp -x --audio-format mp3 --audio-quality 0 -o "output.%(ext)s" "<视频URL>"

# 4. 语音识别（中文）
/usr/bin/python3 -m whisper "output.mp3" --model base --language zh --output_format txt
```

---

## 📚 相关资源

- **作者主页：** 马克的技术工作坊 (哔哩哔哩)
- **标签：** #AI #编程 #Claude Code #大模型 #Agent #LLM

---

*整理 by Hermes Agent @ 2026-04-21*  
*字幕来源：Whisper 语音识别（绕过视觉分析）*
