# UIGen

AI-powered React component generator with live preview. Describe what you want, and Claude generates production-ready React components instantly.

## Features

- **AI-powered generation** — Claude writes React components from natural language descriptions
- **Live preview** — See your component rendered in real-time as it's being generated
- **Virtual file system** — No files written to disk; everything stays in memory
- **Code editor** — View and edit generated files with syntax highlighting
- **Iterative refinement** — Continue chatting with the AI to tweak your components
- **Anonymous & authenticated** — Use without an account, or sign up to save your projects
- **Export** — Download the generated code when you're done

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 (App Router) |
| UI | React 19, Tailwind CSS v4 |
| Language | TypeScript |
| AI | Anthropic Claude, Vercel AI SDK |
| Database | Prisma + SQLite |
| Transpiler | Babel (in-browser JSX transform) |

## Prerequisites

- Node.js 18+
- npm

## Setup

1. Copy the environment file and (optionally) add your Anthropic API key:

```bash
cp .env.example .env
```

```env
ANTHROPIC_API_KEY=your-api-key-here
```

> **Note:** The app works without an API key. Without one, a mock provider returns static code instead of calling Claude.

2. Install dependencies, generate the Prisma client, and run database migrations:

```bash
npm run setup
```

## Running the Application

```bash
# Development server (with Turbopack)
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

```bash
# Production build
npm run build
```

## Usage

1. Sign up or continue as an anonymous user
2. Describe the React component you want in the chat input
3. Watch the component render in the live preview pane
4. Switch to **Code** view to inspect or edit the generated files
5. Keep chatting to refine or extend the component

## Development

```bash
# Run all tests
npm test

# Run a single test file
npx vitest run src/path/to/file.test.ts

# Lint
npm run lint

# Reset the database
npm run db:reset

# Regenerate Prisma client after schema changes
npx prisma generate

# Create a new migration
npx prisma migrate dev
```

## Architecture Overview

```
User prompt
  → ChatInterface → useChat (Vercel AI SDK) → POST /api/chat
  → Claude streams tool calls (str_replace_editor, file_manager)
  → FileSystemContext updates the in-memory VirtualFileSystem
  → PreviewFrame transpiles JSX with Babel and renders in an iframe
```

- **Virtual File System** (`src/lib/file-system.ts`) — in-memory file tree serialized as JSON in the `Project.data` database column
- **Preview Pipeline** (`src/lib/transform/jsx-transformer.ts`) — client-side Babel transform, import map generation (local blobs + `esm.sh` for third-party packages), and HTML page construction with Tailwind CDN and React 19
- **Auth** (`src/lib/auth.ts`) — JWT sessions stored in cookies; `server-only` keeps auth logic out of the client bundle
- **Database** — Prisma + SQLite with two models: `User` and `Project`

## License

MIT
