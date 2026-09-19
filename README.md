# StealthPay

[![StealthPay](public/social/banner.png)](https://stealthpay.up.railway.app)

## Live

**Web app:** [https://stealthpay.up.railway.app](https://stealthpay.up.railway.app)

Open that URL in a browser, or paste it into Nimiq Pay → Mini Apps.

Private invoices. Not on-chain anonymity. StealthPay is a Nimiq Pay mini app for contractors who need to get paid without putting a public name on the invoice.

The payer sees **name or organization, service rendered, and amount**. The recipient wallet stays off the invoice page. Settlement is a normal NIM (or Polygon USDT) transfer. There is no mixer, no shielded pool, and no custodial hot wallet.

## What it does

- Generate a NIM or USDT invoice (individual or organization, service rendered, amount)
- Share a QR / link; checkout is one confirmation
- Issue a read-only viewing key for accountants (no send)
- Audit ledger of invoices for that key
- Honest copy: private invoices, not zk / not a mixer

## Stack

- Next.js 16 (App Router) + TypeScript
- Prisma + PostgreSQL (schema `stealthpay` on a shared database)
- `@nimiq/mini-app-sdk` for NIM + `window.ethereum` / viem for USDT on Polygon (`0x89`, 6 decimals)
- Tailwind v4, Archivo Black + IBM Plex Mono + Mulish
- Official Nimiq dark canvas (`#1F2348` radial) and gold CTAs (`#E9B213`)
- Phosphor icons, mobile-first `min-h-[100dvh]`, 375px WebView

## Shared database (no conflicts with Payrun)

Both mini apps can use **one** Supabase (or any Postgres) project.

| App        | Schema        | Tables |
|------------|---------------|--------|
| Payrun     | `payrun`      | `handles`, `roster_entries`, `pay_cycles`, `pay_items` |
| StealthPay | `stealthpay`  | `businesses`, `invoices`, `viewing_grants`, `settlements` |

Use the **direct** connection string (port `5432`), not the transaction pooler (`6543`). Set the same `DATABASE_URL` on both apps.

```bash
cp .env.example .env
# DATABASE_URL="postgresql://USER:PASSWORD@HOST:5432/postgres?sslmode=require"
npx prisma migrate deploy
```

StealthPay’s migration is named `20260918120001_stealthpay_init` so it will not collide with Payrun’s row in `_prisma_migrations`.

## Develop

```bash
cd stealthpay
npm install
cp .env.example .env
npx prisma migrate deploy
npm run dev
```

Dev server: `http://localhost:3011` (binds `0.0.0.0` for LAN). Use the Network URL, not `localhost`, from a phone.

1. Phone and computer on the same Wi-Fi.
2. Open Nimiq Pay → Mini Apps → paste `http://192.168.x.x:3011`.
3. If the WebView cannot hydrate, add the LAN host to `allowedDevOrigins` in `next.config.ts`.
4. NIM testing: long-press Settings 10 seconds, switch to Testnet.
5. USDT testing uses Polygon mainnet. Do not send real funds unless you intend to.

## Surfaces

| Route | Purpose |
|-------|---------|
| `/` | Intro |
| `/invoice` | Generate NIM or USDT invoice |
| `/invoice/[id]` | QR + share (amount only) |
| `/pay` | Paste an invoice id |
| `/pay/[id]` | One-tap checkout |
| `/audit` | Issue a viewing key |
| `/audit/view/[token]` | Read-only ledger |

## Production

```bash
npm run build
npx prisma migrate deploy
npm start
```

Live deployment: https://stealthpay.up.railway.app

Railway (and similar) should provide `PORT` and `DATABASE_URL`. The start command runs `prisma migrate deploy` then `next start`.

## Honest limits

Funds go to the recipient wallet. There is no withdraw-from-pool flow. A viewing key can read the invoice ledger; it cannot sign or send.
