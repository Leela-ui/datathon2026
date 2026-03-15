# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

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
- **AI**: OpenAI via Replit AI Integrations (no user API key needed)

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   └── polyhome/           # PolyHome React + Vite frontend
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   ├── db/                 # Drizzle ORM schema + DB connection
│   ├── integrations-openai-ai-server/   # OpenAI server-side client
│   └── integrations-openai-ai-react/    # OpenAI React hooks
├── scripts/                # Utility scripts
├── pnpm-workspace.yaml     # pnpm workspace
├── tsconfig.base.json      # Shared TS options
├── tsconfig.json           # Root TS project references
└── package.json            # Root package with hoisted devDeps
```

## PolyHome App

PolyHome is a premium AI-powered real estate consultant with a multi-agent system and voice interaction:

- **Property Search Agent** — returns structured JSON property listings with INR pricing, images, amenities
- **Property Issue Detector** — analyzes property damage/issues from text + images
- **Tenancy Agreement Expert** — answers tenancy/lease/landlord rights questions
- **Real Estate Query Triage Agent** — routes queries to the right specialist

### UI Features
- Centered chat layout with top navigation bar (no sidebar)
- Dark / Light mode toggle
- Voice input via microphone button (speech-to-text via `/api/polyhome/stt`)
- TTS playback on AI messages (via `/api/polyhome/tts`)
- Premium PropertyCard component with image carousel, INR pricing, amenities grid
- Responsive CSS grid for property results (auto-fit / minmax)
- Quick-start suggestion chips on empty state

### Key API Endpoints

- `POST /api/polyhome/chat` — multi-agent chat (SSE streaming, supports image + voice via base64). Emits `{properties:[...]}` event for property search queries
- `POST /api/polyhome/tts` — text-to-speech (returns mp3 audio)
- `POST /api/polyhome/stt` — speech-to-text (accepts base64 webm audio, returns transcript)
- `GET/POST /api/openai/conversations` — manage conversations
- `GET/DELETE /api/openai/conversations/:id` — get/delete a conversation
- `GET /api/openai/conversations/:id/messages` — list messages

### DB Schema

- `conversations` table: id, title, createdAt
- `messages` table: id, conversationId, role, content, createdAt

### AI Integration

Uses Replit AI Integrations for OpenAI access (no user API key required). Env vars: `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY`.

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references.

- **Always typecheck from the root** — run `pnpm run typecheck`
- **`emitDeclarationOnly`** — we only emit `.d.ts` files during typecheck

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build` in all packages
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly` using project references

## Packages

### `artifacts/api-server` (`@workspace/api-server`)

Express 5 API server. Routes live in `src/routes/` and use `@workspace/api-zod` for validation and `@workspace/db` for persistence.

- Depends on: `@workspace/db`, `@workspace/api-zod`, `@workspace/integrations-openai-ai-server`

### `artifacts/polyhome` (`@workspace/polyhome`)

React + Vite frontend for PolyHome. Premium AI real estate chat interface with:
- Airbnb/Apple/Linear-inspired design, dark/light mode toggle
- Centered chat layout, suggestion chips on empty state
- Voice input (MediaRecorder API → STT), TTS playback on assistant messages
- `PropertyCard` component with `ImageCarousel` (swipe/arrows/dots, hover elevation)
- `PropertyGrid` (CSS auto-fit grid, responsive desktop/tablet/mobile)
- `ThemeContext` at `src/context/ThemeContext.tsx` (localStorage persisted)
- framer-motion animations, react-markdown with remark-gfm

### `lib/api-spec` (`@workspace/api-spec`)

Owns the OpenAPI 3.1 spec (`openapi.yaml`) and the Orval config. Run codegen: `pnpm --filter @workspace/api-spec run codegen`

### `lib/db` (`@workspace/db`)

Database layer using Drizzle ORM with PostgreSQL. Schema: conversations, messages.

Push schema: `pnpm --filter @workspace/db run push`

### `lib/integrations-openai-ai-server` (`@workspace/integrations-openai-ai-server`)

Server-side OpenAI client configured from Replit AI Integrations env vars.
