# Valorant Profile

A single-user personal Valorant profile page. Shows identity, social links, and live competitive stats (rank, recent matches, lifetime stats) pulled from the Henrik Valorant API.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **Frontend**: React + Vite, wouter routing, TanStack Query, shadcn/ui (Radix), Tailwind v4, framer-motion, react-icons
- **API framework**: Express 5 with pino logging
- **Database**: PostgreSQL + Drizzle ORM (single `profiles` table)
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **External API**: Henrik Valorant API (`api.henrikdev.xyz`) — requires `HENRIK_API_KEY` secret

## Artifacts

- `artifacts/valorant-profile/` — React + Vite frontend, served at `/`
- `artifacts/api-server/` — Express API, served at `/api`

## Architecture notes

- **Owner-only writes**: PUT `/api/profile` requires an `x-owner-key` header that matches the `OWNER_KEY` env var (returns 401 otherwise). The frontend caches the passcode in `localStorage` under `valorant-profile.owner-key` and prompts via a passcode dialog when missing or rejected. GET endpoints stay public.
- **Background music**: Profile has `musicUrl` + `musicTitle` (owner-editable in the Edit sheet's `AUDIO` tab). The home page renders an `<AudioHud>` widget bottom-right when `musicUrl` is set: `<audio>` auto-plays muted + looped on mount (browser autoplay policy), and a one-shot `click`/`keydown`/`touchstart` listener auto-unmutes on the first user gesture. The HUD shows a play/pause button, "// NOW PLAYING" + track title, and a mute/unmute toggle. Requires a direct audio file URL (e.g. `.mp3`) — SoundCloud/Spotify/YouTube page links won't play in `<audio>`.
- **Single profile model**: one row in `profiles` table; the GET endpoint auto-creates a default row if none exists
- **Region resolution**: every Valorant endpoint resolves the user's true region via `/account` first, then uses that region for MMR/matches calls (Henrik often returns empty 200s when the wrong region is supplied)
- **Player matching by PUUID**: matches are matched against the player's PUUID, not name#tag, because Henrik sometimes returns empty player name/tag fields
- **Caching**: 60s in-memory cache on Henrik responses keyed by request path with in-flight request de-duplication
- **Error mapping**: 404 → "Account not found", 429 → "Rate limited by upstream API. Try again in a moment.", everything else → 502 "Failed to reach the Valorant API."
- **Map splash images**: hardcoded UUID table for current Valorant maps (Ascent, Bind, Breeze, Fracture, Haven, Icebox, Lotus, Pearl, Split, Sunset, Abyss)
- **Tier name fallback**: hardcoded competitive tier number → name table (0=Unranked, 3-27 = Iron 1 through Radiant) used when Henrik's `currenttierpatched` is missing

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
