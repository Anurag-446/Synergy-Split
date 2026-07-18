# SynergySplit

SynergySplit is a game-theoretic coordination app for shared homes. It makes chores, bills, and cooperation visible without turning a household into a surveillance system. Members earn non-transferable Harmony Tokens for useful, timely contributions; a repeated-game reputation score smooths out one bad week; and a GPT-5.6 Sol mediator drafts private, face-saving reminders.

## What is implemented

- Cinematic 3D product landing page with an interactive Harmony Core and live fairness simulator
- Live household overview with harmony, fairness, contribution, and completion metrics
- Command Center with cooperation risk, completion forecast, intervention queue, member radar, and JSON house-report export
- Daily, weekly, and monthly chore automations that regenerate the next actionable occurrence
- Durable Cloudflare D1 data for members, chores, bills, ledger events, and AI nudges
- Atomic chore-completion and bill-payment actions with duplicate-action protection
- Timeliness-aware chore rewards and amount-neutral bill rewards
- Add-chore, add-bill, member-switching, filtering, activity-feed, and demo-reset flows
- Inspectable Fairness Lab explaining the payoff rules and safety constraints
- GPT-5.6 Sol mediator through the OpenAI Responses API
- Deterministic safety fallback when `OPENAI_API_KEY` is not configured
- Responsive desktop/mobile UI, pointer-reactive depth, and reduced-motion support

## Why the game is fairer than a points leaderboard

Harmony Tokens have no cash value and cannot be traded. They are a short-term cooperation signal.

### Chores

```text
chore reward = effort points × timeliness multiplier

early by 12h or more: 1.25
on time:              1.00
late:                 0.70
```

### Bills

Bill rewards depend on timeliness, not the amount:

```text
3+ days early: 8 HT
on time:       6 HT
late:          3 HT
```

That prevents a wealthier member from buying the top reputation simply by paying the most expensive bill.

### Repeated-game reputation

```text
new reputation = 0.70 × prior reputation + 0.30 × current cooperation
```

The memory term rewards consistency while preventing one missed chore from becoming a permanent label.

### Household fairness

The dashboard converts the Gini coefficient of member token totals into a 0–100 fairness score. The score describes the distribution of visible contributions, not anyone's character.

## GPT-5.6 integration

The server-only `/api/nudge` route calls the OpenAI Responses API with:

```text
model: gpt-5.6-sol
reasoning effort: low
maximum output: 160 tokens
```

The mediator receives only the recipient's display name, the requested tone, one task, its deadline, a coarse balance label, and the house harmony score. Its instruction explicitly forbids shaming, threats, diagnoses, public rankings, private-score disclosure, and financial advice. User-supplied names and task text are treated as inert data to reduce prompt-injection risk.

If `OPENAI_API_KEY` is absent or the model call fails, the route returns a labelled deterministic fallback so every non-AI product flow remains demoable. It never pretends the fallback came from GPT-5.6.

## Architecture

```text
React/Vinext dashboard
        │
        ├── /api/household ── D1 ── chores, bills, members, ledger
        │                         └── fairness + reputation engine
        │
        └── /api/nudge ───── OpenAI Responses API (gpt-5.6-sol)
                                  └── safe local fallback
```

Important source locations:

- `app/synergy-landing.tsx` — cinematic landing page and interactive fairness simulator
- `app/synergy-dashboard.tsx` — dashboard and Command Center product UI
- `app/api/household/route.ts` — household reads and mutations
- `app/api/nudge/route.ts` — GPT-5.6 mediation flow
- `lib/household-store.ts` — D1 initialization, queries, and actions
- `lib/game-engine.ts` — payoff, fairness, and reputation functions
- `db/schema.ts` and `drizzle/` — durable schema and migration

## Local setup

Requirements: Node.js 22.13 or newer.

```bash
npm ci
cp .env.example .env.local
# Add OPENAI_API_KEY to .env.local for live GPT-5.6 nudges
npm run dev
```

The local development server provisions a local D1-compatible database through the Sites/Vinext development environment. The first API request creates and seeds the Sunrise House demo.

Generate a migration after changing `db/schema.ts`:

```bash
npm run db:generate
```

## API summary

### `GET /api/household`

Returns the household snapshot, member metrics, chores, bills, activity, saved nudges, and derived fairness statistics.

### `POST /api/household`

Supported actions:

- `complete_chore`
- `pay_bill`
- `create_chore`
- `create_bill`
- `reset_demo`

### `POST /api/nudge`

Accepts `targetMemberId`, `taskId`, and one of the tones `warm`, `direct`, or `playful`. Returns the message, the actual generator label, and an updated snapshot.

## Three-person, two-day execution split

| Window | Person A — platform | Person B — experience | Person C — game/AI |
| --- | --- | --- | --- |
| Day 1 AM | D1 schema and APIs | Dashboard shell and responsive system | Payoff rules and seeded demo story |
| Day 1 PM | Transactions and deployment | Chores, bills, activity interactions | Fairness metrics and mediator prompt |
| Day 2 AM | Error handling and state tests | Motion, accessibility, mobile polish | GPT-5.6 evaluation and safety cases |
| Day 2 PM | Production verification | Demo recording | Pitch, metrics, and judge Q&A |

## Three-minute demo script

1. Start on Overview and explain the 0–100 Harmony and Fairness indicators.
2. Switch the active member, complete an overdue chore, and show the live token/activity update.
3. Open Bills and point out that reputation ignores the bill amount.
4. Open Fairness Lab and explain the repeated-game memory rule.
5. Generate a private mediator nudge and show the real model/fallback label.
6. Reset the demo house for the next judge.

## Scope and safety

This build coordinates a small, consent-based household. It is not a credit score, payroll system, financial ledger, or tool for landlords and employers. A production expansion should add invitations, household-level authorization, audit/export controls, deletion workflows, explicit nudge consent, rate limits, and evaluation with real shared-home participants before real-world use.
