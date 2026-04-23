# Uwa~Nile

## Overview

Uwa~Nile is an African super-app with a global soul. "Global brain, African heart and mind."
- The internal AI agent is **Amamhie Mmadu** ("Knowledge of Humanity" in Igbo)
- Built by **Odoh Tochukwu Peter** (Nwaodoh Tobechukwu / Nwaodoh)

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **Frontend**: React + Vite
- **AI**: OpenAI via Replit AI Integrations (gpt-5.2 for Amamhie Mmadu AI agent)

## Artifacts

- **Uwa~Nile** (`artifacts/uwa-nile`) — React + Vite web app at `/`
- **API Server** (`artifacts/api-server`) — Express API at `/api`
- **Uwanile Mobile** (`artifacts/wafid`) — Expo React Native app (slug: wafid, port 21377)
  - Single-file app in `app/(tabs)/index.tsx`
  - 5 bottom tabs: Amamhie (AI chat), Soro (messaging), Waka (routes), Ogo (marketplace), Owa (tools hub)
  - Colors: AmamhieBlue #2A6AF5, AmamhieGold #FFB800, bg #F5F7FF
  - Floating Amamhie FAB on all tabs except Amamhie tab
  - Other tabs can pre-fill Amamhie chat input and switch to Amamhie tab

## Authentication

Clerk Auth is integrated throughout the app:
- **Provider**: Clerk (white-label, keys auto-provisioned via Replit)
- **Frontend**: `@clerk/react` ClerkProvider in App.tsx
- **Backend**: `@clerk/express` clerkMiddleware + requireAuth for protected routes
- **Sign-in page**: `/sign-in` — branded Clerk component with Uwa~Nile theme
- **Sign-up page**: `/sign-up` — branded Clerk component
- **Protected routes**: `/sura` requires authentication; unauthenticated users redirected to /sign-in
- **User identity**: Display name pulled from Clerk profile (fullName > firstName > username)
- **Layout**: Shows authenticated user's name and sign-out button in sidebar
- **ENV vars**: `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY`, `VITE_CLERK_PUBLISHABLE_KEY`

## Features (Phase 1)

- **Uwa** — AI-curated social feed with African-themed trending posts
- **Amamhie Mmadu** — Internal AI agent with deep African cultural knowledge, multilingual (Igbo, Yoruba, Hausa, Swahili, Zulu, Pidgin, etc.), can translate/transliterate African languages
- Navigation stubs for: Mirroring (video call), Nkiri (live stream), Ije (ride), Ogige (location), Enyi (relationships)
- All feature names in African languages with English subtitles

## Navigation Labels (African Languages)
- **Uwa** = World (Igbo) — Social feed
- **Amamhie** = Knowledge of Humanity (Igbo) — AI agent
- **Mirroring** — Video calls
- **Nkiri** = Watch/View (Igbo) — Live streaming
- **Ije** = Journey (Igbo) — Ride hailing
- **Ogige** = Arena/Meeting Place (Igbo) — Location sharing
- **Enyi** = Friend/Elephant (Igbo) — Relationship AI

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally
- `pnpm --filter @workspace/uwa-nile run dev` — run frontend locally

## Sura — Communication (Requires Auth)

Sura is the communication hub (named "Face" in Swahili). Authentication required.
- **Mirroring** — Peer-to-peer video calls via WebRTC. Optional private rooms (invite-only by Clerk userId). Room codes shared out-of-band.
- **Calls** — Audio-only rooms via WebRTC
- **Messages** — Persistent threaded chat backed by PostgreSQL, real-time updates via SSE
- Private mirror rooms: host specifies allowed Clerk userIds; enforced in WebSocket signaling
- Room metadata stored in `artifacts/api-server/src/lib/roomMeta.ts` (in-memory map)
- WebSocket signaling at `/api/ws`

## Navigation Labels (Current)
- **Luga** = Language (Swahili) — AI language translator
- **Cowrie** — Barter & trade marketplace
- **Cryer** — Notifications
- **Bond** — Bonding (placeholder)
- **Ubuntu** — Relationships (placeholder)
- **Hapa** = Here (Swahili) — Location (placeholder)
- **Waka** = Go (Swahili) — Ride-hailing (placeholder)

## Luga — Language Translator
AI-powered translator with 33 languages and African specialisation.
- Route: `/luga` → `artifacts/uwa-nile/src/pages/luga.tsx`
- API: `POST /api/translate`, `GET /api/translate/languages`
- Route handler: `artifacts/api-server/src/routes/translate/index.ts`
- 33 languages: African-focused (Yoruba, Igbo, Hausa, Swahili, Zulu, Amharic, etc.) + global
- Auto-detect source language via GPT-4o JSON mode
- Language preference persisted to `localStorage["uwa_lang_pref"]`
- Translation history in `localStorage["uwa_translate_history"]` (max 10 items)
- Debounced auto-translation (600ms) as user types
- Swap button: enabled when source is explicitly set OR when language has been detected
- Inline Translate button on every feed post (reads user's lang preference)
- Rate limited: 30 requests/min

## Cowrie — Trade & Barter Marketplace
- Route: `/cowrie` → `artifacts/uwa-nile/src/pages/cowrie.tsx`
- Full CRUD for marketplace listings with interest tracking
- DB tables: `cowrie_listings`, `cowrie_interests`

## Verified Users Tick System
- `isVerified` column on `users` table (PostgreSQL)
- `verification_vouches` table with unique(forName, byName)
- POST /users/:name/vouch — verified users only, 10 vouches auto-grants tick
- POST /users/:name/verify — developer-only (DEVELOPER_NAME=Nwaodoh)
- `VerifiedBadge` component in `artifacts/uwa-nile/src/components/verified-badge.tsx`
- Integrated into Search + Cowrie owner names

## Database Schema Tables
- `conversations` — Amamhie Mmadu AI chat conversations
- `messages` — Chat messages within conversations
- `feed_posts` — Uwa social feed posts
- `post_comments` — Comments on feed posts
- `chat_threads` — Sura message threads (participantA, participantB as display names)
- `chat_messages` — Messages within Sura chat threads
- `cowrie_listings` — Cowrie barter marketplace listings
- `cowrie_interests` — Interest tracking for listings
- `verification_vouches` — Vouch records for verified user tick system

## AI Integration
Uses open AI Integrations for OpenAI access (no user API key required).
System prompt defines Amamhie Mmadu's identity: African cultural expert, multilingual, knows it was built by Nwaodoh.
