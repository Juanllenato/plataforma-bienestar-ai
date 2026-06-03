# Modular Architecture

**Plataforma de Bienestar — AI Health Platform**

## Core principle

Every business module is a self-contained capsule that can be enabled or disabled **per tenant** without affecting the rest of the system. This prepares the platform for a B2B model where a gym buys only the training module and a clinic only labs + projection.

## Module hierarchy

Modules form a strict hierarchy. A module may only depend on modules at its level or below — preventing cycles and allowing whole layers to be disabled.

```
Level 0 Foundation    : core, auth, users, events
Level 1 Shared        : subscriptions, notifications, analytics, storage
Level 2 Data          : labs, workouts, nutrition, wellness, progress
Level 3 Intelligence  : knowledge_base, coach, plan_generator
Level 4 Synthesis     : projection, gamification
Level 5 Commercial    : store
```

## Backend module shape

```
apps/backend/app/modules/{module}/
├── config.py        # ModuleConfig: name, flag, dependencies, level
├── interfaces.py    # Public protocols (what the module exposes)
├── models.py        # SQLAlchemy models (private to the module)
├── schemas.py       # Pydantic schemas
├── service.py       # Business logic (implements the interfaces)
├── routes.py        # FastAPI APIRouter
├── events.py        # Events emitted / handled
└── tests/
```

## Inter-module communication — only two ways

**A) Service lookup by interface** (never import another module's implementation):
```python
lab_service: ILabService = ModuleRegistry.get_service(ILabService)
biomarkers = await lab_service.get_latest_biomarkers(user_id)
```

**B) Event bus** (loose coupling):
```python
await EventBus.emit("lab.processed", payload={"user_id": ..., "lab_id": ...})

@EventBus.on("lab.processed")
async def recalculate_projection(payload): ...
```
Events follow `{module}.{action}`: `lab.processed`, `workout.completed`, `user.onboarded`, `payment.confirmed`.

## Feature flags — evaluated per request

Flags are resolved on **every request**, not at startup, enabling hot enable/disable per tenant with no redeploy.

Resolution order:
1. User override (A/B testing)
2. Tenant configuration (B2B)
3. Global module default

## Rules that don't break

1. No module imports another module's code directly — only via registry or events.
2. Each module is built against an **interface**, not an implementation — replaceable.
3. Feature flags evaluate per request, never at startup.
4. Foundations first; every step ships with its tests passing before the next.

## Stack

| Layer | Tech |
|---|---|
| Backend | Python 3.12 · FastAPI · SQLAlchemy 2.0 async · Pydantic v2 |
| Database | PostgreSQL 16 + pgvector · Redis 7 |
| Migrations | Alembic |
| Mobile | React Native · Expo · NativeWind · Reanimated 3 · Skia |
| State | Zustand (client) · React Query (server state) |
| Auth | Supabase Auth |
| AI | LLM coach (Together/OpenAI) · vision · embeddings + pgvector RAG |
| Monorepo | pnpm workspaces |
