<h1 align="center">ShortFilm Agent</h1>

<p align="center">
  <b>AIGC 内容生产 Agent 底座：一套流程引擎，适配不同行业的图像 / 视频生成产品</b>
</p>

<p align="center">
  <b>简体中文</b> · <a href="./README_EN.md">English</a>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.10+-black.svg">
  <img alt="Node.js" src="https://img.shields.io/badge/Node.js-18+-black.svg">
  <img alt="License" src="https://img.shields.io/badge/License-MIT-black.svg">
</p>

---

## ✨ 这是什么

ShortFilm Agent 是一个**可二次开发的 AIGC 项目底座**。用户输入一句需求，系统把它拆成一条多阶段的生产流水线，每个阶段的产出都可以**查看、确认、修改、单项重新生成**，人始终在回路里。

```
文案/剧本 → 角色与场景设定 → 分镜 → 画面 → 视频 → 成片
```

核心特性：

- 🧩 **场景包即产品** —— 一个文件夹就是一个行业场景，换一个场景包就是另一个产品
- 🔄 **多阶段流水线** —— 内置 6 种阶段类型，按场景自由组合
- 👥 **人在回路** —— 每阶段产出可确认、修改、重生成
- 🔌 **模型可插拔** —— 通义 / 火山方舟 / 可灵 / OpenAI / Gemini / DeepSeek 任意切换
- 🚫 **零代码扩展** —— 新增场景通常只需复制场景包、改 YAML 和提示词

内置 5 个开箱即用的场景：

<table>
  <thead>
    <tr>
      <th align="center">内置场景</th>
      <th align="center">输入</th>
      <th align="center">产出</th>
      <th align="center">适用场景</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="left"><strong>AI 短片</strong>（6 阶段）</td>
      <td align="left">一句故事灵感</td>
      <td align="left">多集剧情短片，可逐集确认、修改、重新生成</td>
      <td align="left">品牌故事、微电影、个人 IP 内容孵化</td>
    </tr>
    <tr>
      <td align="left"><strong>电商广告片</strong>（6 阶段）</td>
      <td align="left">商品与卖点</td>
      <td align="left">30–45 秒竖屏带货视频，突出商品卖点与促销氛围</td>
      <td align="left">抖音、淘宝、视频号等平台的商品短视频投放</td>
    </tr>
    <tr>
      <td align="left"><strong>漫剧</strong>（6 阶段）</td>
      <td align="left">世界观与主角设定</td>
      <td align="left">竖屏多集漫画短剧，人物画风与叙事风格均可定制</td>
      <td align="left">网文改编、轻小说漫改、二次元短剧内容创作</td>
    </tr>
    <tr>
      <td align="left"><strong>旅游宣传片</strong>（6 阶段）</td>
      <td align="left">目的地与宣传重点</td>
      <td align="left">45–60 秒目的地宣传片，突出风光与人文特色</td>
      <td align="left">景区宣传、文旅账号日常内容更新与活动造势</td>
    </tr>
    <tr>
      <td align="left"><strong>商品海报</strong>（4 阶段）</td>
      <td align="left">商品与卖点</td>
      <td align="left">一组 3–4 张电商海报，纯图像流程、无需视频素材</td>
      <td align="left">电商主图、大促活动海报、多平台商品素材设计</td>
    </tr>
  </tbody>
</table>

新增一个场景通常**不需要写代码**：复制一个场景包，改 YAML 和提示词即可。需要更复杂的逻辑时，可以给场景加自定义阶段 Agent。

## 🎓 写给 AI 产品经理同学

这个项目是为「基于底座做行业 AIGC 产品」的课程准备的。建议这样使用：

1. **先跑起来**：用 5 个内置场景各生成一次，感受多阶段 + 人在回路的产品形态。
2. **读 [示例场景拆解](docs/examples.md)**：看同一套引擎怎样通过阶段组合和提示词变成不同产品。
3. **跟着 [30 分钟做出你的第一个场景](docs/tutorial-first-scenario.md)**：为你的目标行业（电商、旅游、教育、房产、餐饮……）做一个场景包。
4. **用 AI 编程工具迭代**：项目附带 [CLAUDE.md](CLAUDE.md) / [AGENTS.md](AGENTS.md)，Claude Code、Cursor 等工具读完就知道该改哪里。你的主要工作是**定义场景、写好提示词、验收效果**。

## 🚀 快速开始

### 环境要求

- Python 3.10+（推荐 3.12，用 [uv](https://docs.astral.sh/uv/) 管理）
- Node.js 18+ / npm 9+
- ffmpeg（拼接视频片段）
- 至少一家模型服务商的 API Key（默认配置使用阿里云百炼 DashScope 和火山方舟 Ark）

### 一键安装与启动

```bash
./scripts/install.sh          # Windows: scripts\install.bat
./scripts/dev.sh              # 同时启动后端 :8000 和前端 :3000，Ctrl+C 一起停止
```

打开 http://localhost:3000 ，在「设置」页填写 API Key，回到首页选择场景、输入需求即可开始。

### 手动启动

```bash
# 后端
cd backend
uv sync --python 3.12
cp config.yaml.example config.yaml     # 也可以在前端「设置」页填写 API Key
uv run python api_server.py            # http://localhost:8000

# 前端（新终端）
cd frontend
npm install
npm run dev                            # http://localhost:3000
```

### Docker Compose

```bash
cp .env.example .env                   # 填入 SFA_*_API_KEY
docker compose up --build
```

## 📁 目录结构

```text
.
├── backend/                     # Python FastAPI 后端
│   ├── scenarios/               # ★ 场景包：每个子目录是一个场景（scenario.yaml + prompts/ + agents.py）
│   ├── core/
│   │   ├── scenarios.py         # 场景包加载与校验
│   │   ├── stages.py            # 6 种内置阶段类型
│   │   ├── orchestrator.py      # 工作流引擎：按场景的阶段列表驱动状态机
│   │   └── agents/              # 内置阶段 Agent
│   ├── prompts/                 # 默认提示词（场景包可按文件覆盖）
│   ├── models/                  # 模型服务商客户端 + 统一门面
│   ├── registry/models.yaml     # 模型注册表
│   └── config.py                # 配置：默认值 → config.yaml → SFA_* 环境变量
├── frontend/                    # Next.js 前端（首页场景选择、阶段工作台、设置页）
├── docs/                        # 文档
├── scripts/                     # install / dev / new-scenario 脚本
└── integrations/agent-skill/    # 可选：供外部 Agent（OpenClaw 等）调用的 Skill
```

## 📚 文档

<table>
  <thead>
    <tr>
      <th align="center">文档</th>
      <th align="center">内容</th>
      <th align="center">一句话说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="left"><a href="docs/tutorial-first-scenario.md">30 分钟做出你的第一个场景</a></td>
      <td align="left">手把手教程</td>
      <td align="left">入门必读：跟着教程用 30 分钟做出属于你自己的第一个场景包</td>
    </tr>
    <tr>
      <td align="left"><a href="docs/examples.md">示例场景拆解</a></td>
      <td align="left">5 个内置场景的设计思路</td>
      <td align="left">理解同一套流程引擎是如何变成不同行业的内容生产产品的</td>
    </tr>
    <tr>
      <td align="left"><a href="docs/scenarios.md">场景包参考</a></td>
      <td align="left">scenario.yaml 字段、提示词覆盖、自定义 Agent</td>
      <td align="left">开发场景包过程中可以随时查阅的完整配置速查手册</td>
    </tr>
    <tr>
      <td align="left"><a href="docs/extending.md">二次开发指南</a></td>
      <td align="left">换品牌、加模型、加服务商、替换 Agent</td>
      <td align="left">把底座深度改造成属于自己产品的完整二次开发指南</td>
    </tr>
    <tr>
      <td align="left"><a href="docs/architecture.md">架构</a></td>
      <td align="left">分层、数据流、状态机</td>
      <td align="left">深入了解多阶段流水线、状态机与数据流的完整运转方式</td>
    </tr>
    <tr>
      <td align="left"><a href="docs/configuration.md">配置</a></td>
      <td align="left">配置文件与环境变量</td>
      <td align="left">三层配置合并机制与全部环境变量优先级的详细说明文档</td>
    </tr>
    <tr>
      <td align="left"><a href="docs/api.md">HTTP API</a></td>
      <td align="left">接口说明</td>
      <td align="left">前后端交互细节与外部 Agent 集成方案的完整接口说明</td>
    </tr>
    <tr>
      <td align="left"><a href="CONTRIBUTING.md">贡献指南</a></td>
      <td align="left">开发约定与提交流程</td>
      <td align="left">参与项目贡献之前必读的开发约定、分支管理与提交流程</td>
    </tr>
  </tbody>
</table>

## 🔧 模型与配置

LLM / VLM / 图像 / 视频各环节的模型可以在通义、火山方舟、可灵、OpenAI、Gemini、DeepSeek，以及任意 OpenAI 兼容接口之间切换。密钥统一填写在 `config.yaml` 的 `api_providers.<服务商 id>.<字段>` 下（「设置」页填写的也是它），各服务商的 id 依次为 `dashscope`、`ark`、`kling`、`openai`、`gemini`、`deepseek`、`openai_compatible`：

<table>
  <thead>
    <tr>
      <th align="center">服务商</th>
      <th align="center">主要用途</th>
      <th align="center">密钥字段</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="left">阿里云百炼</td>
      <td align="left">通义千问系列文本模型、通义万相 Wan 图像与视频生成，也是项目默认配置，覆盖从剧本创作、角色设定、分镜设计到成片合成的全部环节</td>
      <td align="left"><code>api_key</code></td>
    </tr>
    <tr>
      <td align="left">火山方舟 Ark</td>
      <td align="left">豆包 Seedream 图像生成、Seedance 视频生成，适合各类广告片与宣传物料的生产制作</td>
      <td align="left"><code>api_key</code></td>
    </tr>
    <tr>
      <td align="left">可灵 Kling</td>
      <td align="left">可灵 AI 视频生成，画面稳定、动态自然，适合剧情镜头、广告镜头与宣传片</td>
      <td align="left"><code>access_key</code><br><code>secret_key</code></td>
    </tr>
    <tr>
      <td align="left">OpenAI</td>
      <td align="left">GPT 系列文本与视觉模型、OpenAI 图像生成，通用能力强、第三方生态完善</td>
      <td align="left"><code>api_key</code></td>
    </tr>
    <tr>
      <td align="left">Google Gemini</td>
      <td align="left">Gemini 系列文本与视觉模型，多模态理解能力强、支持超长上下文窗口</td>
      <td align="left"><code>api_key</code></td>
    </tr>
    <tr>
      <td align="left">DeepSeek</td>
      <td align="left">DeepSeek 文本模型，性价比高，适合批量生成剧本、分镜与营销文案</td>
      <td align="left"><code>api_key</code></td>
    </tr>
    <tr>
      <td align="left">OpenAI 兼容接口</td>
      <td align="left">任意 OpenAI 兼容接口均可接入，比如 vLLM、Ollama 等自建或第三方模型服务</td>
      <td align="left"><code>api_key</code><br><code>base_url</code></td>
    </tr>
  </tbody>
</table>

配置按三层合并：`backend/config.py` 默认值 → `backend/config.yaml`（「设置」页写入的就是它）→ `SFA_*` 环境变量（只在运行时生效，不写回文件）。详见 [docs/configuration.md](docs/configuration.md)。

> ⚠️ `backend/config.yaml` 和 `.env` 已被 git 忽略。**不要把 API Key 提交到仓库。**

## ✅ 测试

```bash
cd backend && uv run --extra dev pytest      # 含场景包格式检查
cd frontend && npx tsc --noEmit && npm run build
```

## 🗺️ 路线图

- [ ] 更多场景包：知识科普、房产展示、餐饮探店
- [ ] 内置阶段组件读取场景的 options，支持更多无代码配置
- [ ] 跨镜头连贯性审查 Agent
- [ ] 多镜头并行生成

## 📄 License

[MIT](./LICENSE)
