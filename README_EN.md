<h1 align="center">ShortFilm Agent</h1>

<p align="center">
  <b>A reusable base for AI short-film creation agents: from one idea to a finished film</b>
</p>

<p align="center">
  <a href="./README.md">简体中文</a> | <b>English</b>
</p>

---

## Overview

ShortFilm Agent turns a one-line idea, a synopsis or a vague concept into an executable film-production pipeline. Every stage produces intermediate assets you can **view, confirm, edit and regenerate**, and the last stage delivers the finished video:

```
Script → Character/Scene design → Storyboard → Reference images → Video clips → Post-production
```

It is built as a base, not a finished product: branding, models, model providers and stage agents are all replaceable through configuration.

- **End-to-end**: one pipeline from script to final cut, with a confirmation stop after each stage.
- **Pluggable models**: DashScope (Qwen / Wan), Volcano Ark (Seedream / Seedance), Kling, OpenAI, Gemini, DeepSeek, and any OpenAI-compatible endpoint.
- **Config-driven extension**: add models in YAML; swap a stage's agent by pointing config at your class.
- **One-click pipelines**: standard short video, digital human, action transfer.
- **Deploy your way**: install script, `scripts/dev.sh`, or Docker Compose.

## Layout

```text
backend/                  FastAPI backend
  api/                    HTTP routers, schemas, services
  core/stages.py          stage registry (order, names, default agents, overrides)
  core/orchestrator.py    workflow engine: state machine, persistence, cross-stage sync
  core/agents/            the six stage agents (AgentInterface subclasses)
  models/                 provider clients + facades (llm/vlm/image/video_client.py)
  registry/models.yaml    model registry
  pipelines/ prompts/ templates/
  config.py               defaults → config.yaml → SFA_* env vars
frontend/                 Next.js frontend (branding in config/brand.ts)
docs/                     architecture, configuration, extending, API, session format
integrations/agent-skill/ optional skill for external agents (OpenClaw etc.), Feishu/WeChat delivery docs
scripts/                  install.sh / install.bat / dev.sh
docker-compose.yml, .env.example
```

## Quick start

Requirements: Python 3.10+ (3.12 recommended, managed with [uv](https://docs.astral.sh/uv/)), Node.js 18+, ffmpeg.

```bash
./scripts/install.sh      # Windows: scripts\install.bat
./scripts/dev.sh          # backend :8000 + frontend :3000
```

Manual:

```bash
cd backend && uv sync --python 3.12 && cp config.yaml.example config.yaml && uv run python api_server.py
cd frontend && npm install && npm run dev
```

Docker:

```bash
cp .env.example .env      # fill in SFA_*_API_KEY
docker compose up --build
```

Open http://localhost:3000. API keys and default models can also be set on the Settings page.

## Configuration

Three layers, later wins: defaults in `backend/config.py` → `backend/config.yaml` (path overridable with `SFA_CONFIG`; written by the Settings page) → `SFA_*` environment variables (runtime only, never written back). Examples: `SFA_DASHSCOPE_API_KEY`, `SFA_ARK_API_KEY`, `SFA_MODEL_LLM`, `SFA_PORT`, `SFA_CORS_ORIGINS`. See [docs/configuration.md](docs/configuration.md).

## Extending

See [docs/extending.md](docs/extending.md): rebranding, adding models, connecting OpenAI-compatible endpoints, adding providers, replacing stage agents. Architecture: [docs/architecture.md](docs/architecture.md). HTTP API: [docs/api.md](docs/api.md).

## Tests

```bash
cd backend && uv run --extra dev pytest
cd frontend && npm run lint && npm run build
```

## License

[MIT](./LICENSE)
