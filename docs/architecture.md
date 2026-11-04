# 架构

ShortFilm Agent 采用 **单编排器 + 多阶段 Agent** 结构：编排器驱动一个 6 阶段状态机，按顺序调度各阶段 Agent；每个 Agent 完成后把产物落盘并等待用户确认，确认后进入下一阶段。

```
┌──────────────┐  HTTP / SSE  ┌──────────────────────────────────────────────────┐
│  Next.js 前端 │─────────────▶│ api/routers  (workflow, sessions, sandbox, ...)   │
│  (frontend/)  │◀─────────────│        │                                          │
└──────────────┘              │        ▼                                          │
                              │ core/orchestrator.py  WorkflowEngine              │
┌──────────────┐              │   状态机 · 会话持久化 · 跨阶段同步 · 取消/进度       │
│ 外部 Agent    │─────────────▶│        │ 按 core/stages.py 的注册表调度              │
│ (可选 Skill)  │              │        ▼                                          │
└──────────────┘              │ core/agents/*  ScriptWriter → CharacterDesigner →  │
                              │   Storyboard → ReferenceGenerator →                │
                              │   VideoDirector → VideoEditor                      │
                              │        │                                          │
                              │        ▼                                          │
                              │ models/{llm,vlm,image,video}_client.py  统一门面    │
                              │        │ 按 registry/models.yaml 的 provider 路由    │
                              │        ▼                                          │
                              │ models/<provider 客户端>  DashScope / Ark / Kling / │
                              │   OpenAI / Gemini / DeepSeek / OpenAI 兼容接口       │
                              └──────────────────────────────────────────────────┘
```

## 分层

| 层 | 位置 | 职责 |
| --- | --- | --- |
| 配置 | `backend/config.py` | 默认值 → `config.yaml` → `SFA_*` 环境变量三层合并；`PROVIDERS` 表声明所有服务商字段和密钥 |
| HTTP | `backend/api/` | FastAPI 路由、请求模型、CORS、`/code` 静态文件 |
| 编排 | `backend/core/orchestrator.py` | 会话状态机、阶段执行、介入（intervention）、进度回调、跨阶段数据同步、磁盘持久化 |
| 阶段注册 | `backend/core/stages.py` | 阶段 id、顺序、展示名称、默认 Agent；`workflow.agents` 替换 |
| 阶段 Agent | `backend/core/agents/` | 每个阶段一个 `AgentInterface` 子类，实现 `process(input_data, intervention)` |
| 模型门面 | `backend/models/*_client.py` | 对 Agent 提供统一的 `LLM` / `VLM` / `ImageClient` / `VideoClient` |
| 模型注册 | `backend/registry/models.yaml` + `models/config_model.py` | 模型 → 服务商、类型、并发、能力；支持本地覆盖与 `custom_models` |
| 一键流水线 | `backend/pipelines/` | 不经逐阶段确认的快捷流程（标准、数字人、动作迁移） |
| 前端 | `frontend/` | 工作台、阶段确认 UI、设置页、沙盒、流水线页；品牌集中在 `config/brand.ts` |

## 阶段与状态

阶段顺序由 `core/stages.py` 的 `STAGE_SPECS` 决定：

| # | id | Agent |
| --- | --- | --- |
| 1 | `script_generation` | `ScriptWriterAgent` |
| 2 | `character_design` | `CharacterDesignerAgent` |
| 3 | `storyboard` | `StoryboardAgent` |
| 4 | `reference_generation` | `ReferenceGeneratorAgent` |
| 5 | `video_generation` | `VideoDirectorAgent` |
| 6 | `post_production` | `VideoEditorAgent` |

每个阶段的状态为 `pending / running / waiting / completed / stopped / error`。Agent 返回：

```python
{"payload": {...}, "requires_intervention": bool, "stage_completed": bool}
```

`requires_intervention=True` 时阶段进入 `waiting`，前端或外部 Agent 展示产物，用户确认后调用 `POST /api/project/{session_id}/continue` 进入下一阶段。

> 编排器中仍有针对具体阶段的跨阶段同步逻辑（例如剧本续写后把新角色同步到角色阶段）。因此替换某阶段的 Agent 时，需要保持该阶段 payload 的结构；增删阶段需要同时调整编排器，属于更深的改造。

## 模型路由

Agent 只依赖四个门面类，并通过模型 id 调用。门面先在模型注册表中查该模型的 `provider`，按服务商选择客户端；未注册的模型名回退到原有的名称匹配规则（如名称含 `gemini` → Gemini）。新增同一服务商的模型只需要写注册表条目。

## 数据与路径

- 产物以相对后端目录的路径（`code/result/...`）记录在会话 JSON 中，并通过 `/code/*` 提供给前端。后端启动时会把工作目录固定为 `backend/`，所以从任何目录启动都能找到产物。
- 会话：`backend/code/data/sessions/{session_id}.json`，格式见 [session_format.md](session_format.md)。
