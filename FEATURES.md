# Features

How the platform works, module by module, and where AI is used. Product/engineering reference — no user data.

## Vision
A preventive-health mobile app that walks a person through their whole journey: define goals → generate a personalized plan (nutrition + training + supplementation) → track progress → project future health → adjust. AI assists at every step, always with the person's safety and limitations first.

Philosophy: **user data → personalized plan → tracking → projection → adjustment.**

## Modules

**Onboarding & Profile** — initial questionnaire (goal, activity, health conditions, dietary restrictions), editable health profile, body-measurements assistant (9 measurements) with step-by-step guidance and history.

**Auth** — email/password or Google, transparent token auto-refresh (including the streaming chat), password recovery; tokens stored securely on device. (Supabase Auth.)

**AI Coach (Chat)** 🤖 — *the AI core.* Streaming conversational coach that **executes actions** via tool calling (see README). Conversation memory + chat history, live working status, mobile-optimized formatting, source citations, scope classifier + anti-jailbreak. Can log meals from a photo inside the chat (photo analyzed in memory, never stored).

**My Plan** 🤖 — generates a 3-pillar plan (nutrition, training, supplementation) respecting allergies/restrictions/medical conditions, with an LLM-written rationale. Generating a new plan deactivates the previous one and feeds the projection.

**Nutrition** 🤖 — daily calories/macros vs goal; log meals by natural-language description or by photo (vision). Review/edit screen before saving.

**Training** — today's workout (sets/reps/suggested weights), active session logging, exercise detail + personal history, alternative exercises, adherence stats.

**Progress** — weight, body-fat %, muscle mass and measurements over time; weight trend with moving average; onboarding baseline; adherence.

**Projection** 🤖 — projects weight, body composition, and biomarkers at 3/6/12 months (current vs projected + confidence). Weekly weight projection toward the declared goal at a **healthy pace** (won't project impossible changes; warns and proposes realistic goals). Simulator for "what if" changes; top-3 highest-impact actions; auto-recalculates on goal/plan change.

**Labs** 🤖 — centralizes blood tests; **parse a lab report from text/OCR** into structured biomarkers via LLM; "my values" with status (low/normal/high/critical); per-biomarker history with trend. Low biomarkers feed supplementation and projection.

**Wellness** — log sleep, water, steps, stress, energy; feeds projection and coach context.

**Gamification** — activity streaks with calendar, unlockable badges, XP and levels.

**Store** — supplement catalog (name, goal, price, SKU), cart, checkout, purchase history; the coach can search real store products and recommend by goal.

**Health-sync** — integrates device health data (steps, etc.) into wellness/progress.

## Cross-cutting principles
- **Personalization** around real goals and data.
- **Safety first** — allergies/restrictions/conditions asked before plans; conflicting items excluded.
- **Privacy** — meal photos analyzed and discarded; health data processed on Western providers.
- **Everything connected** — measurements, nutrition, training, wellness, labs all feed projection and coach context.
- **Educational** — preventive use; does not replace professional medical advice.
