# Contributing to Read Frog

Thank you for your interest in contributing! Read Frog is an open-source, AI-powered language learning browser extension built by the community. Whether you're fixing a bug, adding a feature, improving documentation, or translating the UI — every contribution matters.

By submitting a pull request or any contribution to this repository, you agree that your contribution will be licensed under:

- [GNU GPLv3](./LICENSE), and
- A separate commercial license granted by FEELIO TECHNOLOGIES LTD.

You represent that you have the right to submit the contribution and that it does not infringe any third-party rights. If you do not agree to these terms, please do not submit contributions.

---

## Table of Contents

- [Quick Start](#quick-start)
- [Project Overview](#project-overview)
- [Development Environment](#development-environment)
- [Available Scripts](#available-scripts)
- [Project Architecture](#project-architecture)
- [Code Style & Conventions](#code-style--conventions)
- [Testing](#testing)
- [Commit Messages](#commit-messages)
- [Pull Request Process](#pull-request-process)
- [Troubleshooting](#troubleshooting)
- [Need Help?](#need-help)

---

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/mengxi-ream/read-frog.git
cd read-frog

# 2. Install dependencies (you need Node.js ≥ 22 and pnpm)
pnpm install

# 3. Start the dev server
pnpm dev

# 4. Load the extension in Chrome:
#    - Open chrome://extensions
#    - Enable "Developer mode"
#    - Click "Load unpacked" and select the ./.output/chrome-mv3/ folder
```

> **No `.env` file is required for standard local development.** The extension uses production `readfrog.app` URLs by default. See [Environment Variables](#environment-variables) if you want to override them or point the extension at local services.

---

## Project Overview

Read Frog is a browser extension that helps users learn languages through immersive translation, powered by 20+ AI providers. It supports:

- **Bilingual & Translation-only** page translation
- **Selection translation** toolbar with Explain, Translate, and TTS
- **Context-aware translation** using page content as context
- **YouTube subtitle translation**
- **Text-to-speech** with 150+ voices
- **Batch requests** to save API costs

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | [WXT](https://wxt.dev/) — Next-gen Web Extension toolkit |
| UI | React 19, TailwindCSS v4, Radix UI / Base UI |
| State | Jotai (atoms) |
| AI SDK | Vercel AI SDK (20+ providers) |
| Testing | Vitest, @testing-library/react, jsdom |
| Linting | ESLint (Antfu config) |
| i18n | `@wxt-dev/i18n` + YAML locale files |

---

## Development Environment

### Prerequisites

- **Node.js** `>= 22.0.0` (we recommend using [nvm](https://github.com/nvm-sh/nvm) or [fnm](https://github.com/Schniz/fnm))
- **pnpm** `>= 10.0.0` (install via `corepack enable` or `npm install -g pnpm`)

> We use `pnpm` as the package manager. Do not use `npm` or `yarn` — they will produce incompatible lockfiles.

### Optional: Environment Variables

Create a `.env` file from the example if you need to override runtime URLs:

```bash
cp .env.example .env
```

| Variable | Required | Description |
|----------|----------|-------------|
| `WXT_API_URL` | No | Backend API URL (default: `https://api.readfrog.app`) |
| `WXT_WEBSITE_URL` | No | Official website (default: `https://www.readfrog.app`) |
| `WXT_OFFICIAL_SITE_ORIGINS` | No | Allowed official site origins, comma-separated, no spaces |
| `WXT_AUTH_COOKIE_DOMAINS` | No | Cookie domains for auth, comma-separated, no spaces |
| `WXT_GOOGLE_CLIENT_ID` | Only for production builds | Google OAuth client ID |
| `WXT_POSTHOG_HOST` | Only for production builds | PostHog analytics host |
| `WXT_POSTHOG_API_KEY` | Only for production builds | PostHog API key |

> For normal local development, you can leave all variables commented out.
>
> If you are developing against local packages or local backend services, use `pnpm dev:local`. That switches the extension's default URLs to localhost-oriented values.

---

## Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start dev server for Chrome (hot reload) |
| `pnpm dev:local` | Start dev server using local package aliases and localhost defaults |
| `pnpm dev:edge` | Start dev server for Edge |
| `pnpm dev:firefox` | Start dev server for Firefox (MV3) |
| `pnpm build` | Production build for Chrome |
| `pnpm build:edge` | Production build for Edge |
| `pnpm build:firefox` | Production build for Firefox |
| `pnpm build:analyze` | Production build with bundle analysis |
| `pnpm lint` | Run ESLint |
| `pnpm lint:fix` | Run ESLint and auto-fix issues |
| `pnpm type-check` | Run TypeScript type checking (`tsc --noEmit`) |
| `pnpm test` | Run all tests with Vitest |
| `pnpm test:watch` | Run tests in watch mode |
| `pnpm test:cov` | Run tests with coverage report |
| `pnpm zip` | Create distribution zip for Chrome Web Store |
| `pnpm zip:edge` | Create distribution zip for Edge |
| `pnpm zip:firefox` | Create distribution zip for Firefox |
| `pnpm zip:all` | Create Chrome, Edge, and Firefox zips |

---

## Project Architecture

Read Frog is organized as a **WXT multi-entrypoint browser extension**. If you've never built a browser extension before, the key concept is: different parts of the extension run in different JavaScript contexts and communicate via message passing.

### Directory Structure

```
src/
├── entrypoints/          # Browser extension entrypoints (WXT convention)
│   ├── background/       # Service Worker — the "brain" of the extension
│   │   ├── index.ts        # Main background script initialization
│   │   ├── analytics.ts    # PostHog analytics
│   │   ├── background-stream.ts  # AI streaming via runtime ports
│   │   ├── edge-tts.ts     # Edge TTS message handlers
│   │   ├── proxy-fetch.ts  # Proxy fetch for CORS requests
│   │   ├── translation-queues.ts  # Page & subtitle translation queues
│   │   └── ...
│   ├── host.content/       # Injected into every web page — main translation logic
│   │   ├── index.ts
│   │   └── translation-control/  # Translation state & UI overlay
│   ├── guide.content/      # In-page onboarding / guide entrypoint
│   ├── input-injector.content/  # Rich-text editor injection support
│   ├── interceptor.content/  # Video/player interception helpers
│   ├── selection.content/  # Injected into every page — selection toolbar
│   │   ├── index.ts
│   │   └── selection-toolbar/    # Floating toolbar on text selection
│   ├── subtitles.content/  # Injected into video pages — subtitle translation
│   │   └── platforms/
│   │       └── youtube/    # YouTube-specific subtitle handling
│   ├── popup/              # Click the extension icon → popup UI
│   ├── options/            # Full settings page (chrome://extensions → Details → Extension options)
│   │   ├── pages/          # Settings sub-pages
│   │   └── components/     # Settings-specific components
│   ├── side.content/       # In-page side UI / floating entrypoint
│   ├── sidepanel/          # Extension side panel page shell
│   ├── offscreen/          # Offscreen document (for audio playback in Chrome MV3)
│   └── translation-hub/    # Shared translation hub UI
│
├── components/           # Shared React components (used by popup, options, content scripts)
│   ├── ui/               # Base UI components (Button, Dialog, Toast, etc.)
│   ├── form/             # Form components
│   ├── translation/      # Translation-related components
│   └── ...
│
├── utils/                # Utility functions & business logic
│   ├── atoms/            # Jotai atoms (state management)
│   ├── config/           # Config schema, validation, storage
│   ├── host/             # Content-script utilities (DOM, translation engine)
│   ├── providers/          # AI provider configuration
│   ├── request/            # Request queue & batching
│   ├── server/             # Server-side utilities (TTS, etc.)
│   └── ...
│
├── types/                # Shared TypeScript types
│   ├── config/             # Config types (large, versioned schema)
│   ├── background-stream.ts
│   └── ...
│
├── hooks/                # Custom React hooks
├── locales/              # i18n YAML files (en, zh-CN, ja, ko, etc.)
└── env/                  # Environment variable schemas
```

### Key Concepts

#### 1. Content Scripts vs Background Script

- **Content scripts** (`*.content/`) run inside web pages. They can read and modify the DOM but have limited access to browser APIs.
- **Background script** (`background/`) runs as a Service Worker. It handles API calls, storage, queues, and analytics. It cannot access the DOM.
- **Communication**: Content scripts and background script talk to each other via the message protocol defined in [`src/utils/message.ts`](./src/utils/message.ts).

#### 2. State Management with Jotai

We use [Jotai](https://jotai.org/) (atomic state) instead of Redux/Zustand. Key atoms are in `src/utils/atoms/`:

- `configAtom` — User settings (language, providers, display mode)
- `configFieldsAtomMap` — Derived atoms for specific config fields

```typescript
// Example: Reading config
import { useAtomValue } from "jotai"
import { configAtom } from "@/utils/atoms/config"

const config = useAtomValue(configAtom)
```

#### 3. AI Streaming Architecture

AI text streaming uses Chrome extension **runtime ports** (long-lived connections):

1. Content script opens a port to the background script
2. Background script streams AI response chunks back via the port
3. Content script updates UI incrementally

See [`src/types/background-stream.ts`](./src/types/background-stream.ts) for the stream types.

#### 4. Config Schema Versioning

User config is versioned (currently v72). When you add a new config field:

1. Update the config type in `src/types/config/`
2. Add a migration in `src/utils/config/migration-scripts/`
3. Increment `CONFIG_SCHEMA_VERSION` in `src/utils/constants/config.ts`
4. Add a test example in `src/utils/config/__tests__/example/`

---

## Code Style & Conventions

### ESLint & Formatting

We use [@antfu/eslint-config](https://github.com/antfu/eslint-config) with custom rules. Key points:

- **Double quotes** for strings
- **Semicolons** are handled automatically
- **Import order** is enforced (sorted alphabetically). Run `pnpm lint:fix` before committing — it will fix import order for you.
- **No floating promises** — always handle promise rejections

```bash
# Auto-fix everything (run this before committing)
pnpm lint:fix
```

### Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Components | PascalCase | `SidebarProvider`, `TranslationCard` |
| Hooks | camelCase, prefix `use` | `useTextToSpeech`, `useMobile` |
| Utils | camelCase | `getOrCreateWebPageContext` |
| Constants | SCREAMING_SNAKE_CASE | `CONFIG_SCHEMA_VERSION` |
| Types/Interfaces | PascalCase | `BackgroundStreamTextSerializablePayload` |
| i18n keys | dot notation | `options.translation.mode.title` |

### File Organization

- One component per file (with rare exceptions for closely related sub-components)
- Co-locate tests in `__tests__/` directories or use `*.test.ts` next to the source
- Prefer named exports over default exports
- Keep files under ~300 lines when possible

### TypeScript

- **Strict mode** is enabled — avoid `any`. Use `unknown` with type guards instead.
- Prefer `interface` for object shapes, `type` for unions/aliases.
- Use `satisfies` for config objects to preserve literal types.

### i18n (Internationalization)

All user-facing strings must be translatable. Add keys to `src/locales/en.yml`, then run:

```bash
# WXT will auto-sync keys to other locale files via its i18n module
pnpm dev
```

Use the key in code:

```tsx
import { i18n } from "#imports"

<h1>{i18n.t("options.title")}</h1>
```

---

## Testing

We use **Vitest** with **jsdom** for unit/component tests and **@testing-library/react** for component testing.

### Running Tests

```bash
# Run all tests once
pnpm test

# Run the local validation variant used by hooks/agents
SKIP_FREE_API=true pnpm test

# Run tests in watch mode (useful during development)
pnpm test:watch

# Run with coverage
pnpm test:cov

# Run a specific test file
pnpm test -- src/utils/host/translate/ui/__tests__/spinner.test.ts

# Exclude live external-service tests explicitly
pnpm test -- --exclude="**/free-api.test.ts"

# Run tests whose names match a pattern
pnpm test -- -t "spinner"
```

### Test Environment

- Tests run in a **simulated browser environment** (jsdom), not a real browser
- `console.log/info/warn/error` are silenced by default (see `vitest.setup.ts`)
- `localStorage` and `sessionStorage` are mocked with in-memory implementations
- Browser APIs (`chrome.i18n`, `chrome.identity`, etc.) are mocked via `fakeBrowser` from WXT

### Writing Tests

```typescript
import { render, screen } from "@testing-library/react"
import { describe, expect, it } from "vitest"
import { MyComponent } from "./my-component"

describe("MyComponent", () => {
  it("renders correctly", () => {
    render(<MyComponent />)
    expect(screen.getByText("Hello")).toBeInTheDocument()
  })
})
```

> **Note**: `src/utils/host/translate/api/__tests__/free-api.test.ts` depends on live external translation services. For local validation, prefer `SKIP_FREE_API=true pnpm test` or exclude that file explicitly. CI also excludes it in the default PR workflow.

### Testing Content Scripts

Content scripts depend on DOM APIs that may not exist in jsdom. Use mocks:

```typescript
// Mock browser-specific modules
vi.mock("#i18n", () => ({
  i18n: { t: (key: string) => key },
}))
```

---

## Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Use when... |
|------|-------------|
| `feat` | Adding a new feature |
| `fix` | Fixing a bug |
| `docs` | Documentation only changes |
| `style` | Code style changes (formatting, semicolons, etc.) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Build system or dependency changes |
| `ci` | CI/CD configuration changes |
| `chore` | Maintenance work that doesn't fit the categories above |
| `revert` | Reverting a previous change |
| `i18n` | Translation / internationalization changes |
| `ai` | AI model or provider-related changes |

### Examples

```bash
feat(selection): add copy-to-clipboard button
docs(contributing): add development setup guide
fix(translate): resolve race condition in batch queue
refactor(options): split provider-config-form into smaller components
test(background): add coverage for analytics message handlers
```

> Hooks are split by scope:
>
> - `pre-commit` runs `lint-staged` (currently `pnpm lint:fix` on staged files)
> - `pre-push` runs `lint`, `type-check`, `test`, and extra safety checks such as blocking uncommented `eruda` imports
>
> If a full-repo check is already failing on the current `main` branch for unrelated reasons, do not treat that as a blocker for reviewing an otherwise valid contribution. Call out the pre-existing failure clearly in your PR.

---

## Pull Request Process

### Before You Start

1. **Check existing issues** — Look for open issues or create one to discuss your idea
2. **Claim the issue** — Comment on the issue so others know you're working on it
3. **Fork & branch** — Create a feature branch: `git checkout -b feat/my-feature`

### Making Changes

1. Write clean, well-commented code
2. Add/update tests for your changes
3. Update documentation if needed (README, inline comments, etc.)
4. Add i18n keys for any new user-facing strings
5. Run the local checks that are relevant and currently actionable:

```bash
pnpm lint          # Should pass with 0 errors
SKIP_FREE_API=true pnpm test          # Matches local hook expectations
WXT_SKIP_ENV_VALIDATION=true pnpm build  # Optional but recommended packaging sanity check
pnpm type-check    # Best-effort full-repo check; note unrelated baseline failures if main is not green
```

> `pnpm build` without `WXT_SKIP_ENV_VALIDATION=true` is intended for real production builds and requires the production env vars documented above.
>
> `pnpm type-check` validates the whole repository, not just your changed files. If it currently fails on unrelated existing issues in `main`, make sure your change does not add new type errors and mention the baseline failures in your PR.

### Submitting the PR

1. Fill out the PR template completely
2. Include screenshots or screen recordings for UI changes
3. Link related issues: `Closes #123`
4. Keep PRs focused — one concern per PR is ideal
5. Be responsive to review feedback

### Review Criteria

Maintainers will check:

- [ ] Code follows the project's style guidelines
- [ ] Tests pass and coverage is maintained
- [ ] TypeScript types are correct (no `any` without justification)
- [ ] i18n keys are added for user-facing strings
- [ ] No `eruda` (debug tool) imports left uncommented
- [ ] Changes don't break existing functionality across Chrome, Edge, and Firefox

---

## Troubleshooting

### `pnpm install` fails or hangs

```bash
# Clear pnpm cache and reinstall without rewriting the lockfile
pnpm store prune
rm -rf node_modules
pnpm install --frozen-lockfile
```

### `pnpm dev` shows "Cannot find module '@t3-oss/env-core'" or similar

This can happen if WXT hasn't generated its auto-generated files. Run:

```bash
# WXT_SKIP_ENV_VALIDATION skips env validation during prepare
WXT_SKIP_ENV_VALIDATION=true pnpm postinstall
# Or simply
pnpm dev
```

### Extension doesn't load in Chrome

1. Make sure you selected the `./.output/chrome-mv3/` folder (not the project root)
2. Check for build errors in the terminal
3. Try reloading the extension on `chrome://extensions` or click the reload button

### Tests fail with "Cannot find module 'defuddle/full'"

The `defuddle` package uses subpath exports that may not resolve in Vitest. This is a known limitation — the module is mocked in relevant tests. If you encounter this in new tests, add:

```typescript
vi.mock("defuddle/full", () => ({
  default: class MockDefuddle { parse() { return { content: "" } } },
  createMarkdownContent: (content: string) => content,
}))
```

### `pnpm build` fails with "WXT_GOOGLE_CLIENT_ID is required"

Production builds validate that certain env vars are set. If you only want to verify the build works, skip validation:

```bash
WXT_SKIP_ENV_VALIDATION=true pnpm build
```

> Do not distribute builds made with `WXT_SKIP_ENV_VALIDATION=true`.

### Translation toolbar doesn't appear on some websites

Some sites have strict Content Security Policies (CSP) or use Shadow DOM heavily. Check the browser console for injection errors. If you're debugging translation issues, the `src/entrypoints/host.content/` directory contains the main translation logic.

---

## Need Help?

- **Discord**: [Join our community](https://discord.gg/ej45e3PezJ) for quick questions and real-time help
- **GitHub Issues**: [Open an issue](https://github.com/mengxi-ream/read-frog/issues) for bugs or feature requests
- **Documentation**: [Official Docs](https://www.readfrog.app/docs)
- **Tutorial**: [Getting Started Guide](https://www.readfrog.app/docs)

### Good First Issues

Looking for a place to start? Check issues labeled [`good first issue`](https://github.com/mengxi-ream/read-frog/labels/good%20first%20issue) or [`help wanted`](https://github.com/mengxi-ream/read-frog/labels/help%20wanted). These are typically:

- Adding support for a new language translation (i18n)
- Fixing typos or improving documentation
- Adding unit tests for uncovered code
- Refactoring large components into smaller ones
- Updating AI provider model lists

---

## License

By contributing, you agree that your contributions will be licensed under the [GNU General Public License v3.0](./LICENSE) and a separate commercial license as described above.

---

**Happy coding!** 🐸 We're excited to see what you build.
