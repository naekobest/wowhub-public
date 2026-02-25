# Architecture Overview

## Stack

WoW Hub is a server-rendered SPA built on **Laravel 12** (PHP 8.4) with **Inertia.js v2** and **React 19**. The backend handles all business logic, queue management, and WarcraftLogs API communication. The frontend is a TypeScript React app rendered server-side via Inertia, with no separate API, no JWT, and no state management library needed.

**Database:** PostgreSQL with LIST partitioning by game version. High-volume tables (raids, players, analysis results) are partitioned per expansion, so queries for Vanilla data never touch MoP partitions and vice versa. Analysis result rows are written once. There is no re-analysis and submitting the same report twice is a no-op.

**Auth:** WarcraftLogs OAuth 2.0 (Authorization Code + PKCE). No email/password accounts. Your WCL identity is your identity. OAuth tokens are stored encrypted at rest.

## Queue Architecture

Analysis jobs are expensive. They make multiple GraphQL API calls and run a pipeline of around 15 services per report. To keep Premium users fast regardless of Free tier load, there are four dedicated worker pools:

| Tier | Queue | Workers (prod) | SLA |
|------|-------|----------------|-----|
| Free | `free` | 5 | < 10 min |
| Premium Basic | `premium` | 10 | < 3 min |
| Premium Pro | `premium-pro` | 15 | < 1 min |
| System | `default` | 2 | |

Free and Premium jobs **never share workers**. A spike in free-tier submissions cannot delay paying users.

## WarcraftLogs API Budget

WCL's v2 API has a per-hour point budget. WoW Hub manages this in three tiers:

1. **User has their own WCL v2 API key**: uses their own budget, dispatched to the highest priority queue for their subscription tier
2. **No own key, app budget available**: uses the shared app budget, normal queue priority
3. **No own key, app budget above 70%**: routed to the budget-constrained queue (longer wait, clearly communicated to the user)

Users without their own API key see a persistent prompt to create one at WarcraftLogs settings. It takes 30 seconds and unlocks better queue priority.

**Rate limit handling:** On a WCL 429, the job releases itself back to the queue with the exact retry-after delay and no tries are decremented. Jobs are configured with a 2-hour deadline to safely outlast WCL's 3600-second budget reset cycle without being marked as failed.

## Expansion Detection

WCL report URLs contain a report code but no expansion information. The expansion is always determined from the **Zone ID** returned by the WCL API, not from any URL slug or user input. Zone IDs are mapped to expansions in a config file. This means "Classic" (a slug that changes meaning each year as the progression server moves forward) always resolves correctly.

## Authentication Flow

1. User clicks "Login with WarcraftLogs"
2. OAuth Authorization Code + PKCE flow
3. Access token and refresh token stored encrypted
4. On first login: character sync job dispatched automatically
5. Viewing reports: fully public, no auth required
6. Submitting reports: requires auth

Personal WCL API keys (for own budget usage) are stored separately, also encrypted.
