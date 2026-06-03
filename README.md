<h1 align="center">Plataforma de Bienestar — AI Health Platform</h1>
<p align="center"><i>An AI-powered preventive-health mobile app where users operate the entire product through a conversational AI coach.</i></p>

<p align="center">
  <img src="https://img.shields.io/badge/Type-Engineering%20Case%20Study-6C2BD9?style=flat-square" />
  <img src="https://img.shields.io/badge/AI-Agentic%20Tool%20Calling-0EA5E9?style=flat-square" />
  <img src="https://img.shields.io/badge/Mobile-React%20Native%20%2B%20Expo-22C55E?style=flat-square" />
</p>

> **Engineering-only case study** by [Juan Perez](https://github.com/Juanllenato). No proprietary source code or user data is reproduced here — this documents the *architecture and AI engineering*. Companion to my [PrevenSalud AI CRM case study](https://github.com/Juanllenato/prevensalud-ai-crm).

---

## 🎥 Demo

https://github.com/Juanllenato/plataforma-bienestar-ai/raw/main/media/app-bienestar-demo.mp4

> The agentic AI coach in action — operating the app through conversation.
> ▶ [Download / watch the demo](./media/app-bienestar-demo.mp4)

---

## 30-second pitch

A mobile preventive-health platform built around one idea: **the user shouldn't navigate menus — they should talk to a coach that does the work for them.** The AI coach is not a chatbot bolted onto the app; it's an **agentic orchestrator** that calls 13 internal tools to read the user's profile, log meals from a photo, generate full health plans, project future biomarkers, search a clinical knowledge base (RAG), and search the web — all from one conversation, with streaming responses and safety guardrails.

Underneath it is a **modular, multi-tenant backend** (FastAPI + PostgreSQL/pgvector) designed so a gym can buy only the training module and a clinic only the labs+projection module — feature-flagged per tenant, evaluated on every request.

---

## The AI core: an agentic coach (tool calling)

The conversational coach orchestrates these tools automatically based on intent — streaming, with live status ("Searching the web…", "Checking your measurements…"):

| Tool | What it does | AI |
|---|---|---|
| `web_search` | Searches the web for product reviews/prices, cites sources | ✅ Web Search |
| `deep_research` | Multi-source deep research | ✅ Tavily (optional) |
| `search_knowledge_base` | Clinical KB search (protocols, biomarkers, recipes) | ✅ RAG (embeddings + pgvector) |
| `get_user_profile` | Reads age, weight, goal, conditions | — |
| `get_body_measurements` | Reads measurements + flags if stale (>7 days) | — |
| `update_body_measurements` | Logs weight/measurements **only on explicit instruction** | — |
| `get_nutrition_summary` | Day's calories/macros/adherence | — |
| `search_foods` | Nutritional lookup | — |
| `get_today_workout` | Today's training | — |
| `generate_projection` | Generates/queries the health projection | ✅ Projection engine |
| `recommend_supplement` | Educational supplement info | ✅ RAG |
| `get_protocol_for_symptom` | Intervention protocol for a symptom | ✅ RAG |
| `generate_health_plan` | Full plan respecting goals + medical limits | ✅ Planners + LLM narrative |

**Why this is the interesting part:** the coach *acts* on the system. "Log this plate" (photo) → vision model detects foods → estimates macros → confirms meal type/time → writes the record → comments on goal fit. That's agentic AI grounded in real, access-controlled application state.

### Safety & reliability engineering
- **Scope classifier + hardened system prompt** — anti-jailbreak; off-topic requests get a *specific, reasoned* refusal, not a generic one.
- **Mandatory safety questions before any plan** — allergies, supplement sensitivities, dietary restrictions, medical conditions. **Never** generates a plan assuming no limitations; conflicting items are excluded.
- **Guarded mutations** — the coach won't silently change a weight goal (it would break the plan); it explains and offers to regenerate.
- **Privacy by design** — meal photos are analyzed in memory and **never stored**.
- **LLM fallback** — primary conversational model with provider fallback; prompt caching to cut cost.

---

## AI inventory (what uses AI, exactly)

| Capability | AI tech |
|---|---|
| Conversational coach | Primary LLM (via Together) + OpenAI fallback · streaming · tool calling |
| Scope classifier / refusals | `gpt-4o-mini` |
| Meal logging by description | `gpt-4o-mini` |
| Meal logging by photo | `gpt-4o-mini` (vision) |
| Web search (reviews, prices) | Web Search (+ Tavily optional) |
| Clinical knowledge base | OpenAI embeddings + `pgvector` hybrid search |
| Lab result parsing | LLM (text → structured biomarkers) |
| Plan generation | Planners + LLM narrative |
| Health projection | Predictive models + LLM narrative |

---

## Architecture highlights

**Modular monolith with strict module hierarchy** — each business module is a self-contained capsule, enable/disable per tenant without touching the rest:

```
Level 0 Foundation    : core, auth, users, events
Level 1 Shared        : subscriptions, notifications, analytics, storage
Level 2 Data          : labs, workouts, nutrition, wellness, progress
Level 3 Intelligence  : knowledge_base, coach, plan_generator
Level 4 Synthesis     : projection, gamification
Level 5 Commercial    : store
```

- **No module imports another directly.** Communication is only via a **service registry (interface lookup)** or an **event bus** (`lab.processed`, `workout.completed`, `user.onboarded`…). Replaceable, testable, no cycles.
- **Feature flags evaluated per request** (not at startup) → hot enable/disable per tenant for B2B, with resolution order: user override (A/B) → tenant config → global default.
- **Everything is connected** — measurements, nutrition, training, wellness, and labs all feed the projection engine and the coach's context.

See [`ARCHITECTURE.md`](./ARCHITECTURE.md) and [`FEATURES.md`](./FEATURES.md) for the full breakdown.

---

## Stack

| Layer | Tech |
|---|---|
| Mobile | React Native · Expo SDK 54 · expo-router · Zustand · TanStack Query · Reanimated 3 · Skia · `react-native-sse` (chat streaming) |
| Backend | Python 3.12 · FastAPI · SQLAlchemy 2.0 async · Pydantic v2 · Alembic |
| Data | PostgreSQL 16 + **pgvector** · Redis 7 |
| AI | LLM coach (Together/OpenAI) · `gpt-4o-mini` vision · OpenAI embeddings · RAG · tool calling · prompt caching |
| Auth | Supabase Auth |
| Monorepo | pnpm workspaces |

---

## Why it belongs in an AI portfolio

This isn't "I called an LLM." It's **agentic AI engineering**: tool orchestration over real application state, RAG with hybrid vector+text search, multimodal input (vision), a predictive projection engine, and the unglamorous 80% — safety classification, guarded mutations, per-request feature flags, provider fallback, and privacy-by-design — that decides whether AI ships or breaks.

---

> *Author: Juan Perez — Applied AI Engineer · [github.com/Juanllenato](https://github.com/Juanllenato)*
> *No proprietary source code or user PII is reproduced. Architecture & engineering only.*
