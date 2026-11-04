<h1 align="center">ShortFilm Agent</h1>

<p align="center">
  <b>AI 短片创作 Agent 项目底座：从一句创意到一部成片</b>
</p>

<p align="center">
  <b>简体中文</b> | <a href="./README_EN.md">English</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-purple.svg" alt="Python">
  <img src="https://img.shields.io/badge/Node.js-18+-green.svg" alt="Node">
  <img src="https://img.shields.io/badge/License-MIT-black.svg" alt="License">
</p>

---

## 项目介绍

ShortFilm Agent 是一个可二次开发的 AI 短片 / 短剧创作 Agent 底座。用户给出一句想法、一段梗概或一个模糊概念，系统把它拆成一条可执行的影视生产流水线，每个阶段都产出**可查看、可确认、可修改、可继续生成**的中间资产，最终交付完整成片：

```
剧本策划 → 角色/场景设计 → 分镜规划 → 参考图生成 → 视频生成 → 后期剪辑
```

它被设计成"底座"而不是成品：品牌、模型、模型服务商、阶段 Agent 都可以通过配置替换，不需要改动核心代码。

| 能力 | 说明 |
| --- | --- |
| 全流程生成 | 一条链路打通剧本、角色、分镜、参考图、视频片段与后期剪辑 |
| 人在回路 | 每个阶段完成后等待确认，支持修改、续写、单项重新生成 |
| 多模型可插拔 | LLM / VLM / 图像 / 视频各环节可在通义、火山方舟、可灵、OpenAI、Gemini、DeepSeek 及任意 OpenAI 兼容接口之间切换 |
| 配置驱动扩展 | 新增模型只需写 YAML；替换某阶段的 Agent 只需在配置里指向自己的类 |
| 一键流水线 | 标准短视频、数字人、动作迁移等无需逐阶段确认的快捷流程 |
| 多种部署方式 | 本地脚本一键安装 / `scripts/dev.sh` 一键启动 / Docker Compose |

视频生成阶段支持三种方式：**首帧生视频**（默认，最稳定）、**首尾帧生视频**（衔接更强）、**参考图生视频**（角色/场景一致性更强）。

---

## 目录结构

```text
.
├── backend/                  # Python FastAPI 后端
│   ├── api/                  #   HTTP 层：routers/ 路由、schemas/ 请求模型、services/
│   ├── core/
│   │   ├── stages.py         #   阶段注册表：阶段顺序、名称、默认 Agent、Agent 替换
│   │   ├── orchestrator.py   #   工作流引擎：状态机、会话持久化、跨阶段同步
│   │   └── agents/           #   6 个阶段 Agent（继承 AgentInterface）
│   ├── models/               #   模型服务商客户端 + 统一门面（llm/vlm/image/video_client.py）
│   ├── registry/models.yaml  #   模型注册表（服务商、类型、并发、能力）
│   ├── pipelines/            #   一键流水线
│   ├── prompts/              #   提示词模板
│   ├── templates/            #   HTML 视频模板
│   ├── config.py             #   配置加载：默认值 → config.yaml → SFA_* 环境变量
│   └── config.yaml.example
├── frontend/                 # Next.js 前端（品牌配置在 config/brand.ts）
├── docs/                     # 架构、配置、扩展、API、会话格式文档
├── integrations/agent-skill/ # 可选：供外部 Agent（OpenClaw 等）调用的 Skill，含飞书/微信推送说明
├── scripts/                  # install.sh / install.bat / dev.sh
├── docker-compose.yml
└── .env.example
```

架构说明见 [docs/architecture.md](docs/architecture.md)。

---

## 快速开始

### 环境要求

- Python 3.10+（推荐 3.12，用 [uv](https://docs.astral.sh/uv/) 管理）
- Node.js 18+ / npm 9+
- ffmpeg（拼接视频片段、音视频后处理）

### 方式一：一键安装

```bash
./scripts/install.sh          # Windows: scripts\install.bat
./scripts/dev.sh              # 同时启动后端 :8000 和前端 :3000，Ctrl+C 一起停止
```

安装脚本会检查依赖、安装前后端依赖、构建前端，并把 `backend/config.yaml.example` 复制为 `backend/config.yaml`。设置 `SFA_SKIP_FRONTEND_BUILD=1` 可跳过前端构建。

### 方式二：手动

```bash
# 后端
cd backend
uv sync --python 3.12
cp config.yaml.example config.yaml     # 填入 API Key
uv run python api_server.py            # http://localhost:8000

# 前端（新终端）
cd frontend
npm install
npm run dev                            # http://localhost:3000；生产模式 npm run build && npm start
```

### 方式三：Docker Compose

```bash
cp .env.example .env                   # 填入 SFA_*_API_KEY
docker compose up --build
```

会话和生成产物保存在 `./data/code`，设置页保存的配置在 `./data/config`。

打开 http://localhost:3000，在输入框写下创意即可开始。API Key 和默认模型也可以在前端「设置」页面填写。

---

## 配置

配置按三层合并，后者覆盖前者：

1. `backend/config.py` 中的默认值
2. `backend/config.yaml`（可用 `SFA_CONFIG` 指定其他路径；前端设置页写入的就是这个文件）
3. `SFA_*` 环境变量：只在运行时生效，**不会**写回 `config.yaml`，适合 Docker / CI 注入密钥

常用环境变量：`SFA_DASHSCOPE_API_KEY`、`SFA_ARK_API_KEY`、`SFA_KLING_ACCESS_KEY`、`SFA_OPENAI_API_KEY`、`SFA_MODEL_LLM`、`SFA_PORT`、`SFA_CORS_ORIGINS`。完整列表见 [docs/configuration.md](docs/configuration.md) 和 [.env.example](.env.example)。

只需填写实际选用模型对应的服务商密钥。默认配置下，图像模型 `doubao-seedream-*` 需要 `ark.api_key`，视频与文本模型 `wan*` / `qwen*` 需要 `dashscope.api_key`。

| 服务商 | 配置字段 | 常用用途 |
| :--- | :--- | :--- |
| DashScope（阿里云百炼） | `dashscope.api_key` | 通义千问、通义万相 Wan 图像/视频 |
| 火山方舟 Ark | `ark.api_key` | 豆包 Seedream 图像、Seedance 视频 |
| 可灵 Kling | `kling.access_key` / `secret_key` | 可灵视频生成 |
| OpenAI | `openai.api_key` | GPT 文本/视觉、OpenAI 图像 |
| Gemini | `gemini.api_key` | Gemini 文本/视觉 |
| DeepSeek | `deepseek.api_key` | DeepSeek 文本 |
| OpenAI 兼容接口 | `openai_compatible.api_key` / `base_url` | vLLM、Ollama、OpenRouter 等自建或第三方文本/视觉模型 |

---

## 二次开发

[docs/extending.md](docs/extending.md) 逐步说明了以下扩展方式：

- **换品牌**：后端 `app.name`，前端 `NEXT_PUBLIC_APP_NAME`
- **加模型**：在 `backend/registry/models.local.yaml` 或 `config.yaml` 的 `custom_models` 中声明，无需改代码
- **接入 OpenAI 兼容模型**：配置 `openai_compatible`，再以 `provider: openai_compatible` 注册模型
- **加服务商**：在 `config.py` 的 `PROVIDERS` 中声明字段，实现客户端，在对应门面中路由
- **替换阶段 Agent**：在 `config.yaml` 的 `workflow.agents` 中填写 `模块:类名`

HTTP 接口见 [docs/api.md](docs/api.md)，会话文件格式见 [docs/session_format.md](docs/session_format.md)。

---

## 产物

所有会话元数据与生成产物保存在 `backend/code/`（通过 `/code/*` 静态访问）：

```text
backend/code/
├── data/sessions/        # 会话元数据 (JSON)
├── data/tasks/           # 一键流水线任务
└── result/
    ├── image/<session>/  # 角色/场景素材 + 分镜参考图
    ├── video/<session>/  # 视频片段与成片
    └── script/           # 剧本/分镜数据
```

---

## 测试

```bash
cd backend && uv run --extra dev pytest
cd frontend && npm run lint && npm run build
```

## 路线图

- [ ] 角色级 Agent 互评（编剧 / 导演 / 连贯性审查各自带循环与记忆，互相 review）
- [ ] 独立的跨镜头连贯性 Agent（人物 / 场景一致性审查，可打回重做）
- [ ] 并发分镜 Agent + 汇总（多镜头并行生成）
- [ ] 阶段级插件：在注册表中增删阶段，而不只是替换 Agent

## License

[MIT](./LICENSE)
