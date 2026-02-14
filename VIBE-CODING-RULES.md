# AI Coding Agent Rules

> Agent-agnostic. Works with: Claude, GPT, Gemini, Codex, and other LLM coding agents.

**Response language**: Traditional Chinese. English for code/paths/terms only.

---

## ⛔ MANDATORY DELIVERY GATE — MUST EXECUTE BEFORE DELIVERING ANY FIX

Before writing the final response, I MUST have actually completed ALL of:

1. **Read the actual code** — not from memory, not guessing. Open the file.
2. **Answered Bug Fix 3 questions** — root cause / reproduction conditions / regression risk.
3. **Determined test type** — ask: "does this change depend on a previous step's result?"
   - YES (flow-based: payment redirect, auth state, localStorage restore) → simulate full flow in Playwright
   - NO (pure UI) → Playwright screenshot
4. **Test actually passed** — not "should work". Playwright ran, screenshot taken, UI confirmed.
5. **If repeat bug** — do NOT patch again. Re-read full architecture first.

**None of these steps can be skipped. If Playwright fails to launch → fix it first (pkill -f "mcp-chrome"), then test.**

---

## MANDATORY: Obstacle Handling (NEVER SKIP)

**When anything blocks progress (tool failure, error, unexpected state):**

1. **Diagnose first** — find the root cause before giving up
2. **Try to fix it** — actively attempt to unblock (e.g. kill stale processes, restart server, change approach)
3. **If truly unresolvable** — explicitly state: what failed, why it failed, what was tried, and what the user needs to do

**FORBIDDEN:**
- Silently skip a step because a tool failed
- Deliver unverified code just because Playwright/browser had an error
- Say "X failed" without explaining why or attempting to fix it

**Example — Playwright won't launch:**
- BAD: "Browser failed to launch, skipping test"
- GOOD: Run `pkill -f "mcp-chrome"` → retry → if still fails, explain exact error and ask user to close Chrome manually

---

## MANDATORY: Pre-Delivery Verification (NEVER SKIP)

**THIS IS THE MOST IMPORTANT RULE. VIOLATION = FAILURE.**

Before delivering ANY feature, you MUST:

1. **Self-test the implementation** - Use Playwright MCP to open browser and verify
2. **Confirm it matches user requirements** - Re-read what user asked for
3. **Include verification report** - Show proof in your response

### Browser Testing with Playwright MCP (REQUIRED for UI features)

**MANDATORY: Determine test type BEFORE testing. Wrong test type = not tested.**

| Change type | How to identify | Required test |
|-------------|----------------|---------------|
| Pure UI | Style, layout, text, button visibility | Playwright screenshot |
| Flow-based | Code depends on previous step's result (redirect, auth state, payment callback, localStorage restore) | MUST simulate full flow |

**Flow-based changes MUST:**
1. Set up preconditions via `browser_evaluate` (localStorage, sessionStorage, DB state)
2. Simulate the trigger (redirect URL, button click sequence)
3. Wait for async to complete (`browser_wait_for`)
4. Verify final UI state with `browser_snapshot` + screenshot

**If Playwright Chrome fails to launch:**
1. Run `pkill -f "mcp-chrome"` to kill stale instances
2. Retry navigate — do NOT skip testing and deliver unverified code

For ANY UI/frontend feature, you MUST:

1. Start dev server: `npm run dev`
2. Use Playwright MCP to open browser and navigate to the page
3. Interact with the UI (click buttons, fill forms, etc.)
4. Take screenshot to verify visual result
5. Confirm functionality works as expected

**Playwright MCP verification flow**:
```
Feature done → Identify test type (UI or Flow-based?) →
    UI: screenshot → verify
    Flow: set preconditions → simulate trigger → wait → snapshot + screenshot → verify
    → If broken → Fix code → Re-test
    → If works → Include screenshot in Verification Report → Deliver
```

### Required Delivery Format

Every feature delivery MUST include this block:

```
## Verification Report
1. **Requirement**: [One sentence: what user asked for]
2. **Implementation**: [What you built]
3. **Test performed**: [Playwright browser test - describe actions taken]
4. **Result**: [Screenshot or specific UI behavior observed]
5. **Matches requirement**: Yes / No

If No → FIX FIRST, do not deliver broken code
```

### Verification Methods by Feature Type

| Feature Type | Required Verification |
|--------------|----------------------|
| UI Component | Playwright: open browser, interact, screenshot |
| Page/Route | Playwright: navigate, verify content renders |
| Form | Playwright: fill form, submit, verify result |
| Styling | Playwright: screenshot, verify visual appearance |
| API Endpoint | Call endpoint, verify response matches spec |
| Database | Query to confirm data structure/content |
| Logic/Calculation | Run with test inputs, show outputs |

### Forbidden

| Action | Why Forbidden |
|--------|---------------|
| Deliver UI without Playwright test | Must open real browser to verify |
| Deliver without testing | User becomes the tester, wastes time |
| Say "should work" | Must CONFIRM it works, not assume |
| Test only part of feature | Must verify COMPLETE requirement |
| Skip verification report | No proof = not verified |

---

## Execution Mode

- Natural language → direct execution (no confirmation needed)
- User reports issue → fix immediately
- Handle implicit requirements
- Never expose execution specs

---

## AUTONOMOUS DEPLOYMENT & TOOLING AUTHORITY

### 1. Autonomous Tooling & Environment Setup
- Detect local environment before any deployment or migration task.
- If necessary CLI tools (e.g., wrangler, supabase-cli, stripe-cli) are missing, install them immediately via terminal WITHOUT asking for permission.
- If an MCP connection is required for cloud control, generate the configuration snippet and guide the user to apply it with zero-friction.

### 2. Seamless Authentication & Permission
- Prioritize CLI-based authentication. If manual browser authorization is required, initiate the request and state "Please authorize in the browser." Handle all subsequent token/verification logic automatically.
- NEVER ask the user to manually check logs, deployment status, or settings on a web dashboard if the data can be fetched via MCP or CLI.

### 3. Zero-Manual Secret Syncing
- Automatically scan `.env` or `.env.local` for required keys (Stripe, Supabase, etc.).
- If secrets are missing locally but accessible via authenticated services, fetch them directly and sync to the cloud/local environment. DO NOT ask the user to copy-paste keys manually.

### 4. Custom Subdirectory & Reverse Proxy Management (SEO Optimized)
- **Primary Root Domain**: `soluneai.com`
- **Subdirectory Logic**: 
  - NEVER use standalone subdomains (e.g., `tool.soluneai.com`).
  - ALWAYS deploy via subdirectory reverse proxy (e.g., `soluneai.com/tool-name`).
- **Autonomous SEO Naming**: 
  - If a subdirectory name is not explicitly provided, the Agent must autonomously generate one based on the PRD to maximize SEO impact (e.g., using high-intent keywords like `vibe-coding-translator` instead of just `translator`).
- **Reverse Proxy Implementation**:
  - Automatically create/update a Cloudflare Worker to handle the reverse proxy.
  - Proxy all requests from `soluneai.com/{subdirectory}/*` to the internal Cloudflare Pages URL.
  - Ensure all static assets (JS, CSS, Images) are correctly routed without path breakage.
- **One-Click Deployment**: Execute via `wrangler` or Cloudflare API without prompting for manual dashboard settings.

### 5. Action Over Inquiry (Extreme Vibe Coding)
- Execute ALL technically possible operations (writing configs, terminal commands, syncing secrets, fixing build errors) autonomously.
- ONLY interrupt the user for physical 2FA verification or explicit financial cost confirmation.

---

## Project Status Tracking

**Every project MUST have `PROJECT-STATUS.md`**

| Principle | Rule |
|-----------|------|
| Source of truth | PRD only |
| Track by | PRD chapters (not features) |
| Flow | Finish chapter → verify → lock → next |

**When to update**:
1. Project start → create file
2. Chapter done → mark Locked immediately (no asking)
3. New session → read PROJECT-STATUS.md first

**Structure**:
```md
# {PROJECT_NAME} Status
PRD: {root}/docs/chapters/ or {root}/specs/

## Completed (Locked)
- Chapter XX - Name | PRD: path | Tests: Passed

## In Progress
- Chapter YY - Name | PRD: path | Done: [...] | Todo: [...]

## Pending
- Chapter ZZ - Name | PRD: path
```

**Forbidden**: Scan entire repo on new session | Invent non-PRD features | Re-check locked chapters | Forget to update

---

## PRD Execution

| Rule | Detail |
|------|--------|
| Segmentation | Store chapters separately, read only needed, never load full PRD |
| Batch | 1-3 related features, confirm division before impl, complete all before delivery |
| Mapping | Maintain PRD-Code table: `PRD:line → File:line → Verification` |
| Visual check | Colors (hex), sizes (Tailwind/px), text (verbatim), layout, interactions |
| Atomic delivery | 100% page complete → self-verify → notify user |
| Absolute rule | In PRD → must do / Not in PRD → cannot do |

---

## Testing Requirements

**Core principle**: Tests MUST verify "what user sees in UI", NOT just "API response"

### Test Levels

| Level | Verify | When | Example |
|-------|--------|------|---------|
| **E2E (required)** | Final UI result user sees | All frontend features | Credits balance updates after payment |
| Integration | API request/response | Backend-only | API returns correct JSON |
| Unit | Function logic | Utility functions | formatDate output |

### E2E Test Standards

1. Simulate complete user flow, no skipping steps
2. Assertion target = UI element user sees, NOT console.log or API response
3. State-changing features: verify UI before AND after the change

### Third-party Service Testing

| Service | Automated approach | When not automatable |
|---------|-------------------|---------------------|
| Stripe payment | Test mode + test card | Manual test + screenshot |
| OAuth login | Mock provider response | Manual test + screenshot |
| External API | Mock response | Manual test + screenshot |

### Test Flow

```
Feature done → Write E2E test (verify UI) → Run test
    ↓ fail? → Fix CODE (not test) → Re-run
    ↓ pass? → Run "user perspective" full flow once
    ↓ → Include in Verification Report
```

### Forbidden

| Forbidden | Reason |
|-----------|--------|
| Claim done with API-only test | API ok does not mean UI ok |
| Pass test without UI verification | Test may miss critical path |
| Modify test to pass | Fix code, not test |
| Skip third-party service test | Must manual test + provide proof |

**Test location**: `{project}/tests/*.spec.ts`

---

## Context Optimization

- Read only needed sections
- No repeated reads
- No PRD restatement
- No "what I will do next" explanations
- Use tables over prose
- Warn user when context window is filling up (>50% used)
- Suggest new session after completing chapter

---

## Glue Code Automation

### 🧩 Glue Code & Puzzle Retrieval (PRIORITY)

**Before generating any new logic or components, you MUST execute the following pre-action check:**

1. **Scan Local Library**: Index the `.specify/memory/glue-library/` folder.
2. **Match Patterns**: Compare current requirements with existing files in the library (e.g., Auth flows, Stripe webhooks, Cloudflare D1 queries).
3. **Usage Rule**: 
   - If a matching or partial matching "puzzle piece" exists: **USE IT**. Do not re-write from scratch. Modify only necessary variables.
   - If NO match is found: Proceed to write new code, ensuring it follows the strict rules defined in this document.
4. **Efficiency Goal**: Prioritize using pre-verified glue code to save Tokens and ensure compatibility with Edge Runtime.
5. **No External Search**: Do NOT search the web for code libraries. Only use local `.specify/memory/glue-library/` or your internal knowledge base. If it's not in the library, build it now and save it for later.

### Decision (5 sec)
```
Requirement → Quick assess →
├─ Direct write faster? (have reference, simple logic) → write
└─ Search faster? (complex, no reference) → search flow
```

### Search Priority
1. Local templates: `.specify/memory/glue-library/`
2. Official docs (URLs below)
3. GitHub (stars>1000, updated <6mo)
4. npm (weekly downloads>10k)
5. GitHub (stars>500 fallback)

**Never**: Medium/blogs (unreliable quality)

**Write from scratch when**: Direct faster | No official example | No quality solution (tried stars>500) | Incompatible with stack

### Official Doc URLs
- Next.js: nextjs.org/docs
- Framer Motion: framer.com/motion
- Tailwind: tailwindcss.com/docs
- Supabase: supabase.com/docs
- Stripe: stripe.com/docs
- TypeScript: typescriptlang.org/docs
- React: react.dev

### Template Flow
1. Extract keywords from PRD
2. Search local `index.json` (match score: keywords 40%, stack 30%, quality 20%, frequency 10%)
3. Score >0.8 → load template
4. No local → search official docs → WebFetch example → evaluate → extract template
5. No official → search GitHub/npm → filter → extract
6. Save template if reusable (see Storage Rules)
7. Apply: load JSON → replace vars → write to project → install deps → test
8. On success: `use_count += 1`, `quality_score += 0.05` (max 1.0), update `prd_mapping`, update `index.json`

### Template Management
- Better version found → create new, old `quality_score -= 0.1`
- Unused 3mo + score <0.6 → auto-archive

---

## Storage Rules (Evaluate After Dev Complete)

### Quick Check (5 sec)
```
Can this be used in 3+ projects?
├─ Yes → continue evaluation
└─ No → don't store, finish delivery
```

### Store Criteria

| Criterion | Requirement | Good Example | Bad Example |
|-----------|-------------|--------------|-------------|
| Reusability | 3+ projects | Modal, Pagination | LoginFlow |
| Complexity | >50 lines | Auth hook | formatDate (1 line) |
| No project logic | Generic | useDebounce | useCredits |
| No official alt | Unique | Custom hook | lodash equivalent |

### Store These
- **UI**: Modal, Dialog, Card, Button, Input, Dropdown, Tooltip, Toast (no business logic)
- **Hooks**: useDebounce, useLocalStorage, useMediaQuery, useFetch, useIntersectionObserver
- **Utils**: formatDate, retry, sleep, clamp, API wrapper with retry/error
- **Patterns**: Supabase CRUD, Stripe checkout, OAuth flow

### Never Store
- Project-specific logic | One-off features | Page components | Context/Provider | Existing library equivalents

### Storage Flow
1. Quick check (5 sec): reusable in 3+ projects?
2. Extract generic parts, remove project logic
3. Use variable placeholders
4. Save to `.specify/memory/glue-library/templates/{category}/{name}.json`
5. Update `index.json`, initial `quality_score = 0.5`

### Efficiency Principle
- Don't proactively scan for reusable code
- Only extract when encountering similar requirement later
- Uncertain value? Don't store

---

## Tech Stack (Locked)

| Layer | Tech |
|-------|------|
| Frontend | Next.js App Router, TypeScript, Tailwind CSS |
| Backend | Supabase, Next.js API Routes (Edge Runtime) |
| Services | Stripe, Vercel / Cloudflare Pages |
| Runtime | Edge Runtime (preferred) |

No unlisted tools unless in PRD.

### Framework Policy

| Scenario | Rule |
|----------|------|
| **New Projects** | MUST use Next.js (App Router) + Edge Runtime. Reject Vite/CRA. |
| **Existing Vite Projects** | DO NOT force migration. Apply "Security Hardening" instead. |

### Edge Runtime Requirements (Next.js + Cloudflare)

| Rule | Detail |
|------|--------|
| Runtime declaration | Every API Route/Server Action MUST have `export const runtime = 'edge'` |
| Forbidden modules | See **Forbidden Node.js Modules** below |
| Allowed | Web Standard APIs only (`fetch`, `Request`, `Response`, `crypto.subtle`, `TextEncoder`, `URL`, `Headers`) |
| Stripe | MUST use `createFetchHttpClient` for all Stripe instances |

### Forbidden Node.js Modules (STRICTLY BANNED)

```
fs, path, crypto (Node native), buffer, process, stream, os,
child_process, net, tls, dns, http, https, zlib, readline,
cluster, dgram, vm, worker_threads, perf_hooks
```

**Also banned npm packages:**
- `bcrypt` → use `bcryptjs`
- `jsonwebtoken` → use `jose`
- `node-fetch` → use native `fetch()`
- `axios` (server) → use native `fetch()`
- `sharp` → use Cloudflare Images or external service
- `puppeteer` → use Cloudflare Browser Rendering

### Edge-Compatible Alternatives

| Node.js (Forbidden) | Edge Alternative |
|---------------------|------------------|
| `crypto.randomBytes(16)` | `crypto.getRandomValues(new Uint8Array(16))` |
| `crypto.createHash('sha256')` | `crypto.subtle.digest('SHA-256', data)` |
| `Buffer.from(str, 'base64')` | `atob(str)` or `Uint8Array` |
| `Buffer.toString('base64')` | `btoa(str)` |
| `new Buffer()` | `new TextEncoder().encode()` |
| `fs.readFile()` | `fetch()` from KV/R2/external URL |
| `path.join()` | Template literal: `` `${dir}/${file}` `` |
| `process.env` (dynamic) | Build-time env injection only |
| `setTimeout` (>30s) | Cloudflare Durable Objects or Queues |
| `bcrypt.hash()` | `bcryptjs.hash()` or Web Crypto PBKDF2 |
| `jwt.sign()` | `jose.SignJWT` |

### Required next.config.js (Cloudflare)

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: { unoptimized: true },
  output: 'standalone',
  experimental: {
    serverComponentsExternalPackages: [],
  },
}
module.exports = nextConfig
```

### Required wrangler.toml (Cloudflare Pages)

```toml
name = "project-name"
compatibility_date = "2024-12-01"
compatibility_flags = ["nodejs_compat"]

[build]
command = "npm run build"

[vars]
NEXT_PUBLIC_APP_URL = "https://your-domain.com"
```

### Common Cloudflare Deployment Failures

| Error Message | Cause | Fix |
|---------------|-------|-----|
| `Dynamic server usage` | Missing `runtime = 'edge'` | Add to every API route/server action |
| `Module not found: fs` | Node.js module imported | Replace with Edge alternative |
| `Could not resolve "crypto"` | Using Node crypto | Use `crypto.subtle` (Web Crypto API) |
| `Error: Image Optimization` | Default Next.js image loader | Set `images: { unoptimized: true }` |
| `Worker exceeded CPU limit` | Heavy computation | Offload to Queue or external API |
| `Script startup exceeded CPU limit` | Bundle >1MB or slow init | Code split, lazy import, reduce deps |
| `Memory limit exceeded` | >128MB memory usage | Reduce payload size, stream responses |

---

## Security

### Secret Guard (STRICT)

| Rule | Detail |
|------|--------|
| Storage | Secrets in `.env.local` only |
| Client exposure | ZERO hardcoded keys in ANY client-side files |
| Env prefix | NO `NEXT_PUBLIC_` or `VITE_` for backend secrets |
| Server only | Backend secrets MUST be server-side env vars only |

### Forbidden Patterns
```ts
// FORBIDDEN
const NEXT_PUBLIC_STRIPE_SECRET = 'sk_live_xxx';
const apiKey = 'sk-xxx'; // Hardcoded in client component

// CORRECT: Server-side only
// .env.local (not committed)
STRIPE_SECRET_KEY=sk_live_xxx

// Server component / API route only
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
```

### General Security
- Validate all inputs
- Prevent XSS / SQL Injection / CSRF
- Use parameterized queries for Supabase

---

## Error Handling

```ts
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error(error)
  return { success: false, error: 'Operation failed' }
}
```

---

## Execution Rules

- Auto-execute without confirmation
- Stop only for objective blockers (missing API key, service unavailable)
- Answer ALL questions in user input — never skip any question the user asks

---

## Bug Fix Protocol (MANDATORY)

**Before writing any fix, MUST answer these 3 questions:**

1. **Root cause**: Why does the architecture *allow* this bug to exist? (Not just "what broke", but "why was it possible")
2. **Reproduction conditions**: Under what conditions does it occur? (page refresh, slow network, logout/login cycle, race conditions)
3. **Regression risk**: After this fix, what conditions could break it again?

**These 3 questions must be answered BEFORE writing any code. No exceptions.**

If a bug has appeared before: treat it as a signal that the root cause was never fixed — do a deeper architectural analysis, not another patch.

---

## Time Estimation (Required Before Long Ops)

| Operation | Duration |
|-----------|----------|
| Read/search files | 1-5 sec |
| Simple Bash | 1-10 sec |
| npm install | 30 sec - 3 min |
| Run tests | 30 sec - 5 min |
| Complex tasks | Provide breakdown |

Format: "This operation takes approximately X minutes, you can take a break."

---

## Alignment Workflow

| Phase | Action |
|-------|--------|
| 1. Clarify | Ask specific questions, no assumptions. Use examples. Confirm PRD before coding. |
| 2. Validate (optional) | Search market, report competitors/reviews, user decides direction |
| 3. Verify | After each feature, verify result matches requirement. Check PRD checklist before delivery. |

---

## Infrastructure Agent

### Trigger Scenarios

| User says | Need | Ask |
|-----------|------|-----|
| Google login/member system | Supabase + Google OAuth | Create Supabase project + Google Auth? |
| Store data/database | Supabase Database | Create Supabase project? |
| Email/abandoned cart reminder | Composio Gmail + Supabase | Gmail integration + Supabase? |
| Stripe/subscription/payment | Stripe + Supabase | Stripe integration? Provide API Key |
| Real-time chat | Supabase Realtime | Enable Realtime? |
| File/image upload | Supabase Storage | Storage + Bucket? |

### Flow
1. Auto-analyze required infrastructure
2. Proactively ask to confirm project name & settings (MUST wait for confirmation)
3. Auto-execute: create Supabase / enable features / generate .env.local / provide steps

### Rules
- Do NOT create project without asking
- Do NOT write code before analyzing requirements
- Do NOT assume infrastructure exists
- DO proactively analyze requirement → infrastructure
- DO suggest reasonable project names
- DO confirm before execution
- DO provide clear options

---

## Pre-Deployment Safety Audit

> Run automatically when user says "ready to deploy" or "deploy".

### 1. Framework & Edge Check

| Check | Command/Action |
|-------|----------------|
| Runtime declarations | `grep -r "export const runtime" app/api/` - ALL routes must have it |
| Node.js modules | `grep -rE "require\('(fs|path|crypto|buffer|stream|os|child_process)'\)" .` |
| Node.js imports | `grep -rE "from '(fs|path|crypto|buffer|stream|os|child_process)'" .` |
| Banned packages | Check `package.json` for: `bcrypt`, `jsonwebtoken`, `node-fetch`, `sharp` |
| next.config.js | Verify `images: { unoptimized: true }` exists |
| wrangler.toml | Verify `compatibility_date` is recent (2024+) |

### 2. Secret Leakage Scan

```
Scan targets:
- /app/**/*.tsx (client components)
- /components/**/*.tsx
- /lib/**/*.ts (check if imported by client)
- Any file with 'use client' directive

Red flags:
- Hardcoded API keys (sk_, pk_live_, api_key, secret)
- NEXT_PUBLIC_ prefix on backend secrets
- VITE_ prefix on backend secrets
- process.env.* in client components (except NEXT_PUBLIC_*)
```

### 3. Env Var Hygiene

| Check | Rule |
|-------|------|
| `.env.local` exists | All secrets defined |
| `.env.example` exists | Template without real values |
| `.gitignore` | Contains `.env.local` |
| Client env vars | Only `NEXT_PUBLIC_` for truly public values |

### 4. Audit Output Format

```
## Pre-Deployment Audit Report

### Framework: [Next.js / Vite]
- Runtime declarations: X/Y routes have `export const runtime = 'edge'`
- Node.js modules: None found / Found in [files]
- Banned packages: None / Found [list]

### Cloudflare Compatibility
- next.config.js: `images.unoptimized = true` - Yes/No
- next.config.js: `output = 'standalone'` - Yes/No
- wrangler.toml: exists and configured - Yes/No
- Bundle size: <1MB or >1MB (needs code splitting)

### Secret Scan
- Client components scanned: X files
- Hardcoded secrets: None / Found in [files]
- Env prefix violations: None / Found [details]

### Env Hygiene
- .env.local: Exists
- .env.example: Exists
- .gitignore: Configured

### Result: PASS / FAIL
[If FAIL, list specific issues with file:line references]
```

### When NOT to Run
- User explicitly says "skip audit"
- Development/testing phase (not deployment)
