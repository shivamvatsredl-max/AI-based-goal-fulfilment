# DreamForge — Dream to Reality Goal Engine

## Overview

Full-stack goal achievement app with AI-powered task breakdown, gamification, clan system, and AI coaching.

## Stack

- **Monorepo**: pnpm workspaces
- **Frontend**: React + Vite (Tailwind CSS, shadcn/ui, framer-motion, wouter routing)
- **Auth**: Clerk (`@clerk/react` frontend, `@clerk/express` backend)
- **Backend**: Express 5 (Node.js 24, TypeScript, esbuild bundle)
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **AI**: OpenAI via `@workspace/integrations-openai-ai-server` (model: `gpt-5-mini`)
- **API codegen**: Orval (from OpenAPI spec in `lib/api-spec/openapi.yaml`)

## Packages

- `artifacts/dreamforge` — React+Vite frontend, served at `/`
- `artifacts/api-server` — Express API server, served at `/api`
- `lib/db` — Drizzle ORM schema & migrations
- `lib/api-spec` — OpenAPI spec + Orval config
- `lib/api-client-react` — Generated React Query hooks + custom fetch with Clerk JWT
- `lib/api-zod` — Generated Zod schemas from OpenAPI
- `lib/integrations-openai-ai-server` — OpenAI client via Replit AI integrations

## Database Tables

- `users` — profiles synced from Clerk (id, clerkId, email, displayName, age, goalCategories, isPremium)
- `goals` — user goals (title, description, category, targetDate, status)
- `tasks` — AI-generated micro-tasks per goal (90 tasks per plan, difficulty 1-5, dayNumber 1-90)
- `user_stats` — XP, streaks, task/goal completion counts
- `user_badges` — earned achievement badges per user
- `clans` — community groups (name, category, inviteCode, ownerId)
- `clan_members` — clan membership with roles (owner/member)
- `clan_messages` — clan chat messages

## API Routes

All routes require `Authorization: Bearer <clerk-jwt>` (handled by `requireAuth` middleware).

- `GET/PATCH /api/users/me` — profile management
- `GET/POST /api/goals` — list/create goals
- `GET/PATCH/DELETE /api/goals/:goalId` — goal detail/update/archive
- `POST /api/goals/:goalId/generate-tasks` — AI-generate 90 daily tasks
- `GET /api/goals/:goalId/tasks` — list tasks for goal
- `PATCH /api/goals/:goalId/tasks/:taskId` — mark complete (awards XP + badges)
- `GET /api/dashboard/summary` — dashboard stats (streak, XP, level, quote)
- `GET /api/dashboard/today-tasks` — today's pending tasks across all goals
- `GET /api/dashboard/activity-heatmap` — completion activity by date
- `POST /api/ai/chat` — AI coach chat (goal-aware, conversational)
- `GET /api/ai/weekly-report` — AI-generated weekly progress summary
- `GET /api/gamification/profile` — XP, level, badges, streaks
- `GET /api/gamification/leaderboard` — global leaderboard (top 20)
- `GET/POST /api/clans` — browse/create clans
- `GET /api/clans/my` — current user's clan
- `GET /api/clans/:clanId` — clan detail with members
- `POST /api/clans/:clanId/join|leave` — membership management
- `GET/POST /api/clans/:clanId/messages` — clan chat

## Gamification System

- XP = `task.difficulty * 20` per completed task (difficulty 1-5)
- Levels: Dreamer → Explorer → Builder → Achiever → Champion → Warrior → Master → Legend → Titan → Forge Master
- Badges: First Step, Getting Started, Momentum Builder, Century Club, Week Warrior, Iron Discipline
- Streaks: daily activity tracking with consecutive day counting

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

## Important Notes

- After codegen, `lib/api-zod/src/index.ts` should only export `from "./generated/api"` (not schemas)
- Clerk proxy middleware runs at `/clerk` path on the API server
- `setGetTokenFn` from `@workspace/api-client-react` wires Clerk JWT into all API calls
- The `getLevel()` and `BADGES` are exported from `artifacts/api-server/src/routes/tasks.ts`
