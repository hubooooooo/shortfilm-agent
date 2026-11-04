# 二次开发指南

## 换品牌

- 后端：`config.yaml` 的 `app.name` / `app.tagline`（或 `SFA_APP_NAME`），影响 API 标题和 HTML 视频模板里的品牌文字。
- 前端：`NEXT_PUBLIC_APP_NAME`、`NEXT_PUBLIC_APP_TAGLINE`。默认值在 `frontend/config/brand.ts`。CSS 类名统一使用 `sfa-` 前缀，主题样式集中在 `frontend/app/globals.css`。

## 新增模型（已有服务商）

不需要改代码。在 `backend/registry/models.local.yaml`（不纳入版本控制）或 `config.yaml` 的 `custom_models` 中声明：

```yaml
models:                       # models.local.yaml 需要这一层；custom_models 下直接写模型 id
  qwen3.8-plus:
    name: Qwen 3.8 Plus
    provider: dashscope
    family: qwen
    type: [llm]
    concurrency: 10
```

同名条目会与内置注册表合并，可用于覆盖内置模型的并发数等字段。重启后端或在设置页保存一次配置后生效，然后就能在设置页或创建项目时选择该模型。

图像 / 视频模型需要补充 `capabilities`（支持的比例、分辨率、时长、能力类型），可参考 `registry/models.yaml` 中同服务商的条目。

## 接入 OpenAI 兼容模型

适用于 vLLM、Ollama、OpenRouter、Moonshot 等提供 OpenAI Chat Completions 接口的服务（文本与视觉理解）：

```yaml
api_providers:
  openai_compatible:
    api_key: sk-...
    base_url: http://127.0.0.1:11434/v1

custom_models:
  qwen2.5:14b:
    name: Ollama Qwen2.5 14B
    provider: openai_compatible
    type: [llm]
    concurrency: 2

models:
  llm: qwen2.5:14b
```

## 新增服务商

1. 在 `backend/config.py` 的 `PROVIDERS` 中声明字段，例如 `"myvendor": {"label": ..., "fields": {"api_key": "", "base_url": "...", "enable_proxy": False}, "secrets": ("api_key",)}`。这会自动带来默认配置、密钥脱敏、`Config.MYVENDOR_API_KEY` 属性和 `SFA_MYVENDOR_API_KEY` 环境变量。
2. 在 `backend/models/` 下实现客户端，参考同类型的现有客户端（如 `video_kling.py`）的方法签名。
3. 在对应门面（`llm_client.py` / `vlm_client.py` / `image_client.py` / `video_client.py`）中按 `provider == "myvendor"` 路由到新客户端。
4. 在注册表中以 `provider: myvendor` 注册模型。
5. 如需在设置页编辑，在 `frontend/app/settings/page.tsx` 的 `GROUPS` 中添加字段。

## 替换阶段 Agent

每个阶段的 Agent 都可以换成自己的实现，而不修改编排器：

```python
# my_plugins/script.py（放在 backend/ 下或已安装到环境中）
from core.agents.script_agent import ScriptWriterAgent

class MyScriptAgent(ScriptWriterAgent):
    async def process(self, input_data, intervention=None):
        result = await super().process(input_data, intervention)
        # 在这里后处理剧本……
        return result
```

```yaml
workflow:
  agents:
    script_generation: my_plugins.script:MyScriptAgent
```

要求：类必须继承 `core.agents.base_agent.AgentInterface`，并返回与原阶段相同结构的 payload（编排器的跨阶段同步依赖这些字段）。最稳妥的做法是继承原 Agent 并在其结果上修改。阶段 id 见 [architecture.md](architecture.md)。

## 修改提示词

提示词模板在 `backend/prompts/<阶段>/`，`*_zh.txt` / `*_en.txt` 分别对应中英文。风格预设在 `backend/prompts/style/`。

## 外部 Agent 集成

`integrations/agent-skill/` 是可选的 Agent Skill：让 OpenClaw、Claude Code 等外部 Agent 通过 HTTP API 驱动 6 阶段流程，并在每个确认点停下等用户确认；其中也包含飞书 / 微信消息推送说明。运行本项目不需要它。
