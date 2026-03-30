# CLAUDE.md

This file provides guidance for AI assistants working in this repository.

## Project Overview

A full-stack web application template built with **Bun**, **React 19**, and **Tailwind CSS 4**. Bun serves as the runtime, package manager, bundler, and HTTP server — no Node.js, Webpack, or Vite involved.

## Commands

```bash
bun install          # Install dependencies
bun dev              # Start dev server with HMR (hot module reload)
bun start            # Start production server (NODE_ENV=production)
bun run build.ts     # Build to dist/ for distribution
bun run build.ts --help  # Show all build options
```

There are no test commands — this project has no testing infrastructure.

## Architecture

### Directory Structure

```
/
├── src/
│   ├── index.ts        # Bun HTTP server entry point (backend + frontend serving)
│   ├── index.html      # HTML template (entrypoint for bundler)
│   ├── frontend.tsx    # React app bootstrap (mounts <App/> to DOM)
│   ├── App.tsx         # Root React component
│   ├── APITester.tsx   # Interactive API testing component
│   ├── index.css       # Global styles (Tailwind import + custom CSS)
│   ├── logo.svg        # Bun logo
│   └── react.svg       # React logo
├── build.ts            # Production build script with CLI arg parsing
├── bun-env.d.ts        # Ambient type declarations (SVG, CSS modules)
├── bunfig.toml         # Bun configuration (Tailwind plugin, env prefix)
├── tsconfig.json       # TypeScript config
└── package.json
```

### How the Server Works

`src/index.ts` uses Bun's native `serve()` API. It:
- Serves `src/index.html` for all unmatched routes (`/*`)
- Exposes `/api/hello` (GET and PUT) and `/api/hello/:name` (GET) as example API routes
- Enables HMR and browser console forwarding in development

When adding new API routes, add them to the `routes` object in `src/index.ts` using Bun's route syntax.

### How the Frontend Works

`src/index.html` loads `src/frontend.tsx` as an ES module. Bun's bundler (via `bun-plugin-tailwind`) processes the TypeScript and CSS on the fly in dev, and via `build.ts` for production.

`frontend.tsx` bootstraps React with `createRoot`. `App.tsx` is the root component — it imports `APITester.tsx` and `index.css`.

## Code Conventions

### TypeScript

- **Strict mode enabled** — `strict: true` plus `noUncheckedIndexedAccess`, `noImplicitOverride`, `noFallthroughCasesInSwitch`
- Target: `ESNext`, module resolution: `bundler`
- Path alias `@/*` maps to `src/*` (configured in `tsconfig.json`) — use it for imports within `src/`
- JSX transform: `react-jsx` (no need to import React in component files)

### React Components

- Functional components only
- Named exports for non-root components (e.g., `export function APITester()`)
- Default export for the root `App` component
- Use `useRef` for uncontrolled form elements; avoid controlled inputs unless state is needed elsewhere
- Fetch API for HTTP calls; construct URLs with `new URL(endpoint, location.href)` to handle relative paths correctly

### Styling

- **Tailwind-first** — use utility classes for all styling; avoid inline styles
- Tailwind 4 is imported via `@import "tailwindcss"` in `index.css` — no `tailwind.config.js` needed
- The app uses a **dark theme** by default:
  - Background: `bg-[#242424]`
  - Text: `text-[rgba(255,255,255,0.87)]`
  - Accent/highlight: `#fbf0df`, `#f3d5a3`
  - Code/card backgrounds: `bg-[#1a1a1a]`
- Use arbitrary Tailwind values (`bg-[#hex]`, `drop-shadow-[...]`) for exact design values
- Animations: use `animate-[name_duration_timing_iteration]` syntax

### Environment Variables

- Public env vars must be prefixed `BUN_PUBLIC_` to be accessible in the browser (configured in `bunfig.toml`)
- `process.env.NODE_ENV` is set to `"production"` during builds; check it for dev-only behavior
- No `.env` file exists yet — create one if needed (it is gitignored)

## Build System

`build.ts` wraps Bun's build API with:
- Auto-discovery of all `.html` files in `src/` as entrypoints
- CLI argument parsing supporting all `Bun.BuildConfig` options
- `dist/` cleanup before each build
- Default options: `minify: true`, `sourcemap: "linked"`, `target: "browser"`
- Build metrics output (file sizes, build time)

Override defaults by passing flags: `bun run build.ts --no-minify --sourcemap=none`

## Adding Features

### New API Route

Add to the `routes` object in `src/index.ts`:

```ts
"/api/your-route": {
  async GET(req) {
    return Response.json({ ... });
  },
},
```

### New React Component

Create `src/YourComponent.tsx` with a named export, then import it where needed:

```ts
import { YourComponent } from "./YourComponent";
```

### New Static Asset

Place SVG/image files in `src/`. Import them directly — Bun's bundler handles them, and `bun-env.d.ts` provides type declarations for `.svg` imports.

## Key Dependencies

| Package | Version | Purpose |
|---|---|---|
| `react` / `react-dom` | ^19 | UI rendering |
| `tailwindcss` | ^4.1.11 | Utility CSS |
| `bun-plugin-tailwind` | ^0.1.2 | Tailwind integration with Bun bundler |
| `@types/bun` | latest | Bun runtime type definitions |

## What Does Not Exist (Yet)

- No testing framework or test files
- No linter (ESLint) or formatter (Prettier) config
- No CI/CD pipeline
- No state management library
- No routing library (single-page, no client-side routing)
- No environment-based configuration beyond `NODE_ENV`
