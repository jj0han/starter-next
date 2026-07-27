# starter-next

A production-ready Next.js starter kit for new projects. Clone it or use it as a GitHub template to skip boilerplate and start building features on day one.

## Contents

- [What's included](#whats-included)
- [Getting started](#getting-started)
  - [Use as a GitHub template](#use-as-a-github-template)
  - [First steps in a new project](#first-steps-in-a-new-project)
- [Scripts](#scripts)
- [Project structure](#project-structure)
- [tRPC](#trpc)
- [shadcn/ui](#shadcnui)
- [Theming](#theming)
- [Code quality](#code-quality)
- [Global 404 (experimental)](#global-404-experimental)
- [Proxy (route protection)](#proxy-route-protection)
- [Deployment](#deployment)
- [License](#license)

## What's included

| Layer | Stack |
| --- | --- |
| Framework | [Next.js 16](https://nextjs.org) (App Router, React Server Components, Turbopack dev server) |
| UI | [shadcn/ui](https://ui.shadcn.com) (`base-luma` style, zinc palette, CSS variables) |
| Styling | [Tailwind CSS v4](https://tailwindcss.com) |
| Icons | [Hugeicons](https://hugeicons.com) |
| API | [tRPC v11](https://trpc.io) + [TanStack Query v5](https://tanstack.com/query) |
| Validation | [Zod v4](https://zod.dev) |
| Serialization | [SuperJSON](https://github.com/blitz-js/superjson) |
| Theming | [next-themes](https://github.com/pacocoursey/next-themes) (light / dark / system) |
| Language | TypeScript (strict mode) |
| Linting | ESLint (Next.js + import sorting + consistent type imports) |
| Formatting | Prettier + Tailwind class sorting |

### Pre-configured out of the box

- **End-to-end type-safe API** — tRPC router, API route handler, React Query integration, and server-side prefetch/hydration helpers
- **Full shadcn/ui component library** — 50+ components already installed under `components/ui/`
- **Typography helpers** — hand-maintained `components/ui/typography.tsx` (tweak styles to taste; not installed via the shadcn CLI)
- **Global 404 page** — experimental `app/global-not-found.tsx` with `experimental.globalNotFound` enabled in `next.config.mjs`
- **Dark mode** — system-aware theme toggle wired up on the welcome page
- **Fonts** — Inter (sans), Geist (headings), Geist Mono (code) via `next/font`
- **Developer tooling** — React Query Devtools, ESLint, Prettier, and `typecheck` script
- **Proxy placeholder** — `proxy.ts` stub ready for auth or route protection (Next.js 16 proxy convention)

## Getting started

### Use as a GitHub template

1. Click **Use this template** on GitHub to create a new repository.
2. Clone your new repo and install dependencies:

```bash
git clone git@github.com:jj0han/starter-next.git
cd starter-next
pnpm install
```

3. Start the dev server:

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). The welcome page demonstrates theme switching and a prefetched tRPC query.

### First steps in a new project

After scaffolding from this template, customize these before shipping:

1. **Rename the package** — update `name` in `package.json`
2. **Remove demo code** — replace `components/welcome-card.tsx` and simplify `app/page.tsx`
3. **Tune typography** — edit `components/ui/typography.tsx` to match your type scale and brand
4. **Customize the global 404** — update copy and layout in `app/global-not-found.tsx` (keep a full `<html>` / `<body>` document)
5. **Extend the API** — add routers under `trpc/routers/` and register them in `trpc/routers/_app.ts`
6. **Add environment variables** — create `.env.local` (never commit secrets; `.env*.local` is gitignored)
7. **Configure auth** — wire up your provider and uncomment the stubs in `trpc/init.ts` and `proxy.ts`

## Scripts

| Command | Description |
| --- | --- |
| `pnpm dev` | Start dev server with Turbopack |
| `pnpm build` | Production build |
| `pnpm start` | Serve production build |
| `pnpm lint` | Run ESLint with auto-fix |
| `pnpm format` | Format TypeScript/TSX with Prettier |
| `pnpm typecheck` | Run TypeScript compiler without emitting |

## Project structure

```
starter-next/
├── app/
│   ├── api/trpc/[trpc]/route.ts   # tRPC HTTP handler
│   ├── globals.css                # Tailwind + shadcn theme tokens
│   ├── global-not-found.tsx       # Global 404 (experimental; full HTML document)
│   ├── layout.tsx                 # Root layout, fonts, providers
│   ├── page.tsx                   # Home page (prefetch example)
│   └── providers.tsx              # Theme + tooltip providers
├── components/
│   ├── ui/                        # shadcn/ui components
│   ├── theme-provider.tsx
│   └── welcome-card.tsx           # Demo page (safe to remove)
├── hooks/
│   └── use-mobile.ts
├── lib/
│   └── utils.ts                   # cn() helper
├── trpc/
│   ├── init.ts                    # tRPC context + procedure helpers
│   ├── client.tsx                 # Browser provider + useTRPC hook
│   ├── server.tsx                 # RSC helpers: trpc, prefetch, HydrateClient
│   ├── query-client.ts            # TanStack Query + SuperJSON config
│   └── routers/
│       ├── _app.ts                # Root router
│       └── hello.ts               # Example procedure
├── proxy.ts                       # Route proxy stub (auth, redirects)
├── components.json                # shadcn/ui configuration
├── eslint.config.mjs
├── next.config.mjs                # experimental.globalNotFound enabled
├── postcss.config.mjs
└── tsconfig.json
```

Path alias `@/*` maps to the project root.

## tRPC

This template uses the [tRPC + TanStack Query integration for Next.js App Router](https://trpc.io/docs/client/tanstack-react-query/setup).

### Server Components — prefetch data

```tsx
import { HydrateClient, prefetch, trpc } from "@/trpc/server"

export default function Page() {
  prefetch(trpc.hello.list.queryOptions({ text: "world" }))

  return (
    <HydrateClient>
      <MyClientComponent />
    </HydrateClient>
  )
}
```

### Client Components — consume data

```tsx
"use client"

import { useQuery } from "@tanstack/react-query"
import { useTRPC } from "@/trpc/client"

export function MyClientComponent() {
  const trpc = useTRPC()
  const { data } = useQuery(trpc.hello.list.queryOptions({ text: "world" }))

  return <p>{data?.greeting}</p>
}
```

### Add a new procedure

1. Create a router in `trpc/routers/`:

```ts
import z from "zod"
import { baseProcedure, createTRPCRouter } from "../init"

export const usersRouter = createTRPCRouter({
  getById: baseProcedure
    .input(z.object({ id: z.string() }))
    .query(({ input }) => ({ id: input.id, name: "Jane" })),
})
```

2. Register it in `trpc/routers/_app.ts`:

```ts
export const appRouter = createTRPCRouter({
  hello: helloRouter,
  users: usersRouter,
})
```

Types flow automatically to client and server — no code generation step.

## shadcn/ui

Components live in `components/ui/` and are configured via `components.json`:

- **Style:** `base-luma`
- **Base color:** zinc
- **Icon library:** hugeicons
- **RSC:** enabled

### Custom styles

Don't want the default look? Design your theme at [ui.shadcn.com/create](https://ui.shadcn.com/create) — pick the style, base color, radius, fonts, icon library, and other options in the visual editor. When you're done, copy the CLI command from the create page and run it in this project.

The create page offers three ways to apply a preset to an existing project:

| Option | What it updates | CLI |
| --- | --- | --- |
| **Full preset** | Theme tokens, fonts, `components.json`, and all detected UI components | `npx shadcn@latest apply <preset>` |
| **Theme only** | CSS variables in `app/globals.css` and related theme config — components stay as-is | `npx shadcn@latest apply <preset> --only theme` |
| **Fonts only** | Font setup in the layout — everything else stays as-is | `npx shadcn@latest apply <preset> --only font` |

You can also combine partial options, e.g. `--only theme,font`. Use the exact preset code or command shown on the create page.

No need to re-run `init` or reinstall components when you only want a new theme or fonts — that's what the `--only` flag is for. Choose **full preset** only when you want every installed component restyled to match.

### Icons

This template ships with [Hugeicons](https://hugeicons.com). Icon library changes are **not** part of partial preset apply — shadcn only supports `--only theme` and `--only font`, not icons. Switching icon packages in [ui.shadcn.com/create](https://ui.shadcn.com/create) or in `components.json` also does **not** retroactively update icons already in the codebase.

If you change icon libraries, you'll need to manually update icon imports and usage in:

- `components/ui/*` — every pre-installed component that references icons
- Your own components — e.g. `components/welcome-card.tsx`

New components added via `npx shadcn@latest add` will use the configured library; everything already installed is on you.

### Adding more components

```bash
npx shadcn@latest add dialog
```

### Using components

```tsx
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { TypographyH1, TypographyLead } from "@/components/ui/typography"
```

### Typography (hand-maintained)

`components/ui/typography.tsx` is **not** from the shadcn registry — it was added manually as a starting point for headings, body copy, and muted text. Change the Tailwind classes in that file however you like; the welcome page uses `TypographyH1`, `TypographyLead`, and `TypographyMuted` as examples.

Exports: `TypographyH1`–`TypographyH4`, `TypographyP`, `TypographyLead`, `TypographyLarge`, `TypographySmall`, `TypographyMuted`, `TypographyBlockquote`, `TypographyInlineCode`.

### Pre-installed components

accordion, alert, alert-dialog, aspect-ratio, avatar, badge, breadcrumb, button, button-group, calendar, card, carousel, chart, checkbox, collapsible, combobox, command, context-menu, dialog, direction, drawer, dropdown-menu, empty, field, hover-card, input, input-group, input-otp, item, kbd, label, menubar, native-select, navigation-menu, pagination, popover, progress, radio-group, resizable, scroll-area, select, separator, sheet, sidebar, skeleton, slider, sonner, spinner, switch, table, tabs, textarea, toggle, toggle-group, tooltip, typography

## Theming

Theme tokens are defined in `app/globals.css` using OKLCH color values. Dark mode is class-based via `next-themes`.

Toggle theme in client components:

```tsx
import { useTheme } from "next-themes"

const { theme, setTheme } = useTheme()
setTheme(theme === "dark" ? "light" : "dark")
```

Use semantic tokens (`bg-background`, `text-foreground`, `border-border`, etc.) instead of hard-coded colors so components stay consistent across themes.

## Code quality

- **ESLint** — Next.js core web vitals, TypeScript rules, automatic import sorting, and enforced `import type` for type-only imports
- **Prettier** — 2-space indent, no semicolons, double quotes, Tailwind class sorting via `prettier-plugin-tailwindcss`
- **TypeScript** — strict mode with path aliases

Run before committing:

```bash
pnpm lint && pnpm typecheck
```

## Global 404 (experimental)

This template enables Next.js [`global-not-found`](https://nextjs.org/docs/app/api-reference/file-conventions/not-found#global-not-foundjs-experimental) so unmatched URLs render a dedicated 404 instead of composing one from the root layout.

**Config** (`next.config.mjs`):

```js
experimental: {
  globalNotFound: true,
}
```

**File:** `app/global-not-found.tsx` — must return a full HTML document (`<html>` and `<body>`). Import `globals.css` and fonts here; this route bypasses `app/layout.tsx`.

The starter ships a minimal page using the `Empty` component. Customize copy, branding, and navigation to fit your app.

**Caveats:**

- Still **experimental** — API may change before it stabilizes.
- Use a plain `<a href="/">` for home links — `next/link` does not work inside `global-not-found` (see comment in the template file).
- Route-level `not-found.tsx` files remain the right choice for segment-specific 404s; this file only handles URLs that do not match any route.

Test locally by visiting a path that does not exist (e.g. [http://localhost:3000/does-not-exist](http://localhost:3000/does-not-exist)).

## Proxy (route protection)

`proxy.ts` is a stub for Next.js route proxy logic — uncomment and extend it for authentication, role checks, or redirects. Example use cases are commented in the file (e.g. protecting `/admin` routes).

## Deployment

Works on any platform that supports Next.js (Vercel, Docker, Node.js). The tRPC client auto-detects `VERCEL_URL` for server-side requests in production.

Build and run locally:

```bash
pnpm build
pnpm start
```

## License

Private starter kit — adjust the license when you publish your fork.
