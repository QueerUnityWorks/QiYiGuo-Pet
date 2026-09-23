# 奇异果宠物 · Qi Yi Guo Pet

一只住在你 Windows 桌面上的 AI 宠物「奇异果助手」。它以一个透明、无边框、始终置顶的桌面窗口常驻在桌面上，可以陪你聊天、答疑、写代码、整理灵感，并根据回复内容自动切换表情。

基于 Unity 2D 开发，AI 对话对接 **Anthropic 兼容 Messages API**（默认使用 DeepSeek）。

---

## ✨ 功能特性

- **桌面宠物形态**：透明背景、无边框、始终置顶的桌面窗口，可拖动，透明区域支持鼠标点击穿透（Windows 打包后生效）。
- **AI 对话**：通过 Anthropic 兼容接口与 AI 模型聊天，支持上下文人设。
- **表情差分**：根据 AI 回复内容自动切换 4 种表情——待机 / 思考 / 疑惑 / 抱歉。
- **上下文记忆**：自动携带最近 20 轮对话作为上下文，保持连续对话。
- **角色人设可配置**：通过 ScriptableObject 配置 AI 的身份、性格、说话风格、能力边界与禁止行为。
- **本地持久化**：API Key、接口地址与对话记录保存到本地 `user-data.json`，重启后自动恢复。
- **多分辨率预设**：内置 HD（1280×720）、Full HD（1920×1080）、2K（2560×1440）窗口预设。
- **UI 透明度可调**：背景图与窗体整体透明度均可在 Inspector 中配置。

---

## 🛠 技术栈 / 环境要求

| 项目 | 说明 |
| --- | --- |
| 引擎 | Unity **2022.3 LTS**（当前使用 `2022.3.62f3c1`） |
| 渲染 | 2D（`com.unity.feature.2d`） |
| UI | uGUI + TextMesh Pro |
| 网络 | `UnityEngine.Networking`（UnityWebRequest） |
| 平台 | Windows Standalone（透明无边框窗口仅 Windows 打包后生效） |

---

## 📂 目录结构

```
Assets/
├── Scenes/                  # 场景
│   └── MainScene.unity      # 主场景
├── 脚本/                    # C# 脚本（按模块划分）
│   ├── AI/                  # AI 请求、配置、角色与事件总线
│   ├── AI上下文记忆/         # 上下文构建（最近 N 轮对话）
│   ├── 对话系统/            # 消息模型、输入/显示控制、历史对话 UI
│   ├── 形象控制脚本/         # AI 形象表情切换
│   ├── UI控制/              # UI 触发、分辨率、透明度
│   ├── 用户数据/            # 本地 JSON 读写
│   ├── 窗口设置/            # Windows 原生透明/无边框/置顶窗口
│   └── 组件脚本/            # 全局配置、配置面板、滚动列表
├── AI角色设定/
│   └── QIYIGUO.asset        # 「奇异果助手」人设 ScriptableObject
├── 图片素材/
│   ├── 奇异果素材/           # 4 种表情精灵图
│   └── Airi GUI/            # UI 素材
├── 字体/                    # 中文字体
└── TextMesh Pro/            # TMP 资源
```

---

## 🚀 快速开始

1. 使用 **Unity Hub** 以 Unity **2022.3 LTS** 打开本项目。
2. 打开场景 `Assets/Scenes/MainScene.unity`。
3. 配置 AI 接口（见下方「配置说明」）。
4. 点击播放即可与「奇异果助手」对话。

---

## ⚙️ 配置说明

### API Key

推荐通过**环境变量**配置，避免密钥被提交进项目：

- 环境变量名：`ANTHROPIC_AUTH_TOKEN`

也可以在运行后的配置界面中输入 API Key 并点击应用，程序会将其保存到本地 `user-data.json`。

### 接口地址与模型

在 `AISettings` 组件（Inspector）中配置，默认值：

| 配置项 | 默认值 |
| --- | --- |
| Base URL | `https://api.deepseek.com/anthropic` |
| Model | `deepseek-v4-pro[1m]` |

请求会发送到 `<Base URL>/v1/messages`，并携带 `x-api-key`、`Authorization: Bearer` 与 `anthropic-version: 2023-06-01` 请求头。任何兼容 Anthropic Messages API 的服务都可替换使用。

### AI 回复格式

系统提示词要求模型**只返回一个 JSON 对象**：

```json
{
  "reply": "回复给用户的话",
  "expression": "Idle | Thinking | Confused | Apology"
}
```

`expression` 决定角色表情，模型未返回时会根据回复文本中的关键词自动推断。

### 角色人设

人设存放在 `Assets/AI角色设定/QIYIGUO.asset`，字段包括身份、性格、说话风格、与用户的关系、能力边界、禁止行为，以及自定义系统提示词（`useCustomSystemPrompt`）。

---

## 🏗 架构说明

项目采用「事件总线 + 单一职责组件」的轻量结构，脚本通过 `static event` 解耦：

- **AIEventBus / GlobalConfigEventBus / DialogueEventBus**：负责模块间消息传递（配置刷新、鉴权变更、分辨率变更、对话历史变更）。
- **AIChatClient**：监听用户消息事件，构建并发送 Anthropic Messages 请求，解析回复。
- **AIContextBuilder**：读取对话记录，组装最近 20 轮上下文。
- **MessageController / MessageData**：输入输出与对话数据的增删。
- **AgentImagerController**：根据解析出的表情枚举切换形象精灵图。
- **UserData**：启动时读取、退出时保存本地 `user-data.json`。
- **WindowSettings**：调用 Win32 API 实现透明、无边框、置顶、可拖动与透明区域点击穿透（仅 Windows Standalone 生效）。

---

## 🖥 构建

- 目标平台选择 **Windows, Mac, Linux**（`Windows`）下的 Standalone 构建。
- 透明无边框、窗口置顶、点击穿透等效果**仅在 Windows 打包后生效**，Editor 中会用预览背景色替代。

---

## 📝 说明

- 当前为第一版，仅支持聊天对话功能。
- 项目中图片/字体素材的版权归其原作者所有（如 `Airi GUI` 素材，详见其附带的 `License` 文件）。
