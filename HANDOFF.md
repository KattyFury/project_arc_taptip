# HANDOFF: TapTip

Fast tips and lì xì (Tet gifts) on Arc. **Email + 6-digit OTP login**, wallet hidden behind Circle Developer-Controlled Wallets (server signs, no user confirmation), the app pays gas. Top requirement: **speed**.

History: [docs/archive/handoff-through-2026-09-24.md](docs/archive/handoff-through-2026-09-24.md) (Vietnamese).

## Key facts

- Split on 2026-08-22 from [`KattyFury/build-on-arc`](https://github.com/KattyFury/build-on-arc); the guide series stays there.
- Original docs (PRD, discovery, wireframes): [`docs/`](docs/).
- Code: [`app/`](app/), originally forked from `circlefin/arc-p2p-payments`, now Next.js 16 + Cloudflare D1/KV + Circle Developer-Controlled Wallets. Supabase and Modular Wallets are gone.
- **Passkey is a local app lock only** (WebAuthn via `@simplewebauthn`), unrelated to transaction signing: `components/app-lock-gate.tsx`, `lib/auth/applock.ts`, `lib/db/applock.ts`.
- **Design source of truth: Figma file `rLGoWK4AHhqov9CKHXJqqE`**, at read time. Re-run `get_design_context` to compare. `design_handoff_taptip/` and `TapTip Design Spec.dc.html` are outdated.
- Live: **https://taptip.fun** on Cloudflare Workers via `@opennextjs/cloudflare`. `workers.dev` is off; add `"workers_dev": true` to `wrangler.jsonc` to re-enable it.

## Where we stopped (09-24b: UI rebuilt from the updated Figma)

Compared 14 Figma frames; **Figma | App** images in `Desktop\taptip-figma-09-24\`. `tsc` and `npm run build` clean. **Not deployed at that handoff: waiting for owner approval of the images.**

**Design:** margin 25 px, width 340; button row y=738 (Back 100 + gap 8 + primary 232), single button 274 centered; Quicksand only; muted `#AEAEAE`, secondary `#686868`, red `#E12B32`, ink `#1A1A1A`; notice cards green `#AFF4C6` / red `#FCB3AD` / yellow `#FFE8A3`; yellow buttons have a 1 px black border; inputs radius 8. Figma assets in `public/figma/`.

**Decisions the owner made:**
- Splash only during real loading. Logged in with Passkey → Splash + Passkey popup before Home. Unlock remembered for the session; hidden **< 5 minutes** does not relock.
- `AppLockGate` wraps all of `/dashboard/*`. Server check: unlock sets KV `applock_unlocked:<sessionToken>` for 30 minutes; otherwise `/api/tip`, `/api/applock/disable` and adding a passkey return 423 `APPLOCK_REQUIRED`.
- Passkey reset after failed unlock: confirmation → `/api/applock/reset` clears the passkey + **locks send/withdraw 24 h** (KV `wallet_lock:<userId>`); viewing and receiving still work.
- Back = previous screen everywhere (`router.back()` with fallback), except Turn on Passkey (no Back). Skipped users are re-prompted at most once a day.
- **QR is TapTip-only:** `taptip:0x…?chain=5042002&currency=USDC`; the Send Tip scanner rejects anything else. Deposit/Withdraw handle the outside world.
- Send Tip: 2×2 amounts ascending, cheapest preselected, EDIT → picker → SAVE; sends immediately after scan; "Tipped $X" popup 3 s; returns to Get Tip when locked or hidden. No fractional tips ($1–$200).
- Withdraw: confirmation box, up to 2 decimals, MAX, always keeps **$0.1** for gas, recorded as `kind=withdraw` (migration `0006_transactions_kind.sql`).
- History: Received / Tipped / Withdrew / Deposited. Deposits come from Blockscout `token-transfers` via `lib/deposits.ts`.
- Setting: Daily tip limit (tips only, enforced in `/api/tip`) + Currency USD/USDC, stored in KV `prefs:<userId>`. No new D1 columns.
- Balance keeps "$" (local currency via oracle later). Log out asks for confirmation.

**Later (owner's call):** display names (needs a database change).

**Still open:** not tested on a real phone; the black square at the Figma menu button is treated as a placeholder (hamburger icon).
