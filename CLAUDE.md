# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Initial setup (install deps + generate Prisma client + run migrations)
npm run setup

# Development server (with Turbopack)
npm run dev

# Build for production
npm run build

# Lint
npm run lint

# Run all tests
npm test

# Run a single test file
npx vitest run src/path/to/file.test.ts

# Reset database
npm run db:reset

# Regenerate Prisma client after schema changes
npx prisma generate

# Create new migration
npx prisma migrate dev
```

## Environment

Copy `.env` and set `ANTHROPIC_API_KEY`. Without it, the app uses a mock provider that returns static code instead of calling Claude.

## Architecture

**UIGen** is a Next.js 15 App Router application that lets users describe React components in a chat, which Claude generates in a virtual file system and renders live in an iframe.

### Request flow

1. User types a prompt → `ChatInterface` → `useChat` (Vercel AI SDK) → `POST /api/chat`
2. API route (`src/app/api/chat/route.ts`) streams a response from Claude (`claude-sonnet-4-5` via `@ai-sdk/anthropic`) with two tools: `str_replace_editor` and `file_manager`
3. Tool calls are streamed back to the client and handled by `FileSystemContext.handleToolCall`, which mutates the in-memory `VirtualFileSystem`
4. `PreviewFrame` watches `refreshTrigger` from `FileSystemContext`, rebuilds an import map using `@babel/standalone` to transpile JSX/TSX on the client, and sets `iframe.srcdoc` to the generated HTML

### Virtual file system

`VirtualFileSystem` (`src/lib/file-system.ts`) is an in-memory tree that never writes to disk. It serializes to/from `Record<string, FileNode>` for transport (stored as JSON in the `Project.data` column).

- The AI uses `str_replace_editor` (create/str_replace/insert/view commands) and `file_manager` (rename/delete) to manipulate files
- `FileSystemContext` wraps the VFS and exposes React state + `handleToolCall` to bridge AI tool calls to the VFS

### Preview pipeline

`src/lib/transform/jsx-transformer.ts` runs entirely in the browser:
1. `transformJSX` — Babel transforms JSX/TSX to plain JS, strips CSS imports, collects import paths
2. `createImportMap` — builds a `<script type="importmap">` that maps local files to blob URLs and third-party packages to `esm.sh`; generates placeholder stub modules for missing imports
3. `createPreviewHTML` — wraps everything in an HTML page with a Tailwind CDN, React 19 from `esm.sh`, and a React error boundary

### Auth

JWT-based, cookie-stored sessions (`src/lib/auth.ts`). The `server-only` import prevents auth code from being bundled on the client. Anonymous users can chat without signing in; their work is tracked via `anon-work-tracker` in `localStorage`.

### Database

Prisma with SQLite (`prisma/dev.db`). Prisma client is generated into `src/generated/prisma/`. Schema: `User` (email/password) and `Project` (name, messages JSON, data JSON, optional userId).

### Contexts

- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) — owns the VFS instance, selected file, and tool-call dispatch
- `ChatContext` (`src/lib/contexts/chat-context.tsx`) — wraps Vercel AI SDK's `useChat`, serializes the VFS on each request, dispatches tool calls to `FileSystemContext`

### Key path aliases

`@/` maps to `src/` (configured in `tsconfig.json`).

### Tests

Vitest with jsdom + React Testing Library. Tests live in `__tests__/` subdirectories next to the code they test.
