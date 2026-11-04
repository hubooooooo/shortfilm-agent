# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

ShortFilm Agent is a reusable base for AI short-film creation agents. A user idea goes through 6 stages — script → character/scene design → storyboard → reference images → video generation → post-production — with a user confirmation stop after each stage.

- `backend/`: Python FastAPI (uv-managed, Python 3.12)
- `frontend/`: Next.js 16 + React 19 + Tailwind 4
- `integrations/agent-skill/`: optional skill for external agents driving the HTTP API (not needed to run the app)
- `docs/`: architecture, configuration, extending, API, session format

Backend runs at `http://localhost:8000`, frontend at `http://localhost:3000`.

## Commands

```bash
./scripts/dev.sh                                  # start both
cd backend && uv run python api_server.py         # backend only
cd backend && uv run --extra dev pytest           # backend tests
cd frontend && npm run dev                        # frontend dev server
cd frontend && npm run lint && npm run build      # frontend checks
curl http://localhost:8000/api/health
```

## Architecture

- `backend/config.py`: config = defaults → `config.yaml` (`SFA_CONFIG`) → `SFA_*` env vars. Env overrides are never written back to the file. `PROVIDERS` declares every model provider's fields and secrets; `Config.<PROVIDER>_<FIELD>` attributes are generated from it.
- `backend/core/stages.py`: stage registry (`WorkflowStage`, `STAGE_ORDER`, `STAGE_SPECS`). `workflow.agents` in config swaps a stage's agent (`module:Class`, must subclass `AgentInterface`).
- `backend/core/orchestrator.py`: `WorkflowEngine` — state machine, session persistence (`backend/code/data/sessions/`), cross-stage sync. Still contains stage-specific sync logic, so replacement agents must keep each stage's payload shape.
- `backend/core/agents/`: six `AgentInterface` subclasses with `process(input_data, intervention)` returning `{"payload", "requires_intervention", "stage_completed"}`.
- `backend/models/{llm,vlm,image,video}_client.py`: facades used by agents. They route by the model's `provider` in the registry and fall back to name heuristics for unregistered models.
- `backend/registry/models.yaml`: model registry, merged with `registry/models.local.yaml` and `config.yaml` `custom_models` (`models/config_model.py`).
- Artifacts are stored as paths relative to `backend/` (`code/result/...`) and served at `/code/*`; the server pins its cwd to `backend/`.
- Frontend branding lives in `frontend/config/brand.ts` (`NEXT_PUBLIC_APP_*`); CSS classes use the `sfa-` prefix.

## Conventions

- Brand-neutral: never hard-code a product name; use `settings.APP_NAME` (backend) or `brand.name` (frontend).
- New config knobs go into `DEFAULT_CONFIG` / `PROVIDERS` in `config.py` and `config.yaml.example`, with an `SFA_*` env var where useful.
- Prompts live in `backend/prompts/<stage>/` (`*_zh.txt` / `*_en.txt`).
