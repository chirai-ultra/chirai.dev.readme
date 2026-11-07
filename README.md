# Chirai

**Describe your business. Open it tested.** — [chirai.dev](https://chirai.dev)

Chirai builds websites, shops, booking sites, phone apps and browser games from a plain description, and tests
them before the owner sees them: every page opened, every button clicked, two test accounts trying to break each
other's data, a test order and booking placed, and (for games) the game played. The owner then runs the business
from Manage site: orders, stock, returns, bookings, WhatsApp and email updates, payments, marketing, data feeds.

This repository is the whole of Chirai: the build engine, the API, chirai.dev and the Studio. It is private.
The engine version is in [`services/compiler/src/version.ts`](services/compiler/src/version.ts) and is
reported by `/health`.

## What a build produces

| Kind | What the owner gets |
|---|---|
| Website / shop / bookings | Next.js + TypeScript site, its own Postgres schema with row-level access rules, sign-in, Manage site, payments (Razorpay, PayU, Stripe, cash on delivery), emails, SEO, published to its own address on Vercel |
| Phone app | One Expo / React Native app for Android and iPhone, its own database and sign-in, Expo Go preview, submission to Google Play and the App Store |
| Game | A Vite + Phaser (2D) or Three.js (3D) browser game with the game kit (scores, leaderboard, sound, touch), played by the Test Agent |

How the kind is decided: [`services/compiler/src/build-kind.ts`](services/compiler/src/build-kind.ts).

## Layout

```
apps/
  web/        chirai.dev — marketing site, docs (src/content/docs/*.md), build page, Studio, Manage site
  api/        the engine's HTTP API: builds, previews, checks, Test Agent, site services, payments, stores
services/
  compiler/   the build pipeline: request → plan → design → code → kits → checks → repair
  mobile-generator/, deployer/
packages/
  cody/       Chirai Cody (code review) and Chirai Pulse (website check) — published to npm as chirai-cody
  model-router/  which AI model does which task ("Chirai ultra" = Claude, "Chirai brain" = open models)
  ir-schema/, types/, logger/
infrastructure/ops/   backup.sh (nightly, 14 days), restore-site.sh, health.sh
scripts/chirai/       check-server.sh, security-check.sh, collect-site.sh, cleanup.ts, showcase-shots.ts
```

## Run it locally

```bash
npm install
cd apps/api && npx tsx src/index.ts      # the engine, port 3001
cd apps/web && npm run dev               # chirai.dev, port 3000
```

Settings come from `.env.local` (see `check-env.ts` for what's required). Never commit it:
`bash scripts/chirai/security-check.sh` finds keys in tracked files.

## Checks before a release

- `npx tsc --noEmit -p apps/web` must be clean; the API's known type errors must not grow.
- Self-tests: `npx tsx services/compiler/src/build-kind.ts --build-kind-self-test` (and `game-kit`, `site-prompt`,
  `design-study`, `apps/api/src/game-test.ts`), `cd packages/cody && npx tsx src/selftest.ts`.
- `NEXT_PUBLIC_SUPABASE_URL=… NEXT_PUBLIC_SUPABASE_ANON_KEY=… npx next build` in `apps/web`.
- On the server after deploying: `bash scripts/chirai/check-server.sh`.

## Deploying

Server: `cd ~/chirai && git checkout -- package-lock.json && git pull && npm install && pm2 restart all`.
chirai.dev redeploys on Vercel after a push. Database changes ship as `chirai-v*.sql` files run once.

## Security

Report problems to the address on [chirai.dev/security](https://chirai.dev/security).
