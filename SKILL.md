---
name: liqpad
description: Deploy Liquid Protocol tokens on Base paired with DIEM using the dynamic-fee hook, then check and claim Liquid creator/LP fees. Use when the user says use liqpad to deploy, use liqpad to launch, pair with DIEM, check liquid fees, claim liquid fees, or mentions factory 0x04F1a284168743759BE6554f607a10CEBdB77760. Do NOT use Bankr Doppler/Clanker launch for these phrases.
tags: [liquid, diem, base, tokens, fees, deploy, launch, bankrbot]
version: 1
visibility: public
metadata:
  clawdbot:
    emoji: "💧"
    homepage: "https://app.liquidprotocol.org/docs"
    requires:
      packages: [liquid-sdk, viem]
---

# Liquid DIEM Launcher for BankrBot

This skill launches tokens through **Liquid Protocol on Base**, not Bankr Doppler.

Signer is always the **Bankr wallet** of the user who mentioned @bankrbot or spoke in Terminal.
RPC is `BASE_RPC` env var if set, otherwise the default public Base RPC (`https://mainnet.base.org` or viem `base` default).
Never read or ask for a private key. Never use `privateKeyToAccount`.

## When to load

Load this skill if the message contains any of:
- use liqpad to deploy / use liqpad to launch
- pair DIEM / pair $DIEM / paired with diem
- dynamic fees / HOOK_DYNAMIC_FEE
- liquid fees / claim liquid / liquid status
- factory `0x04F1a284168743759BE6554f607a10CEBdB77760`

If the user only says "deploy a token" with no Liquid/DIEM/liqpad words, do **not** use this skill. Use Bankr native launch.

## Defaults

See `references/addresses.md`.

- Chain: Base `8453`
- Factory: Liquid `0x04F1a284168743759BE6554f607a10CEBdB77760`
- Hook: Dynamic Fee V2
- Locker: LP Locker Fee Conversion
- MEV: Sniper Auction V2
- Quote: DIEM `0xf4d97F2da56e8c3098f3a8D538DB630A2606a024`
- Tick spacing: 200
- Starting cap helper: $20,000 USD using live DIEM USD price
- Dynamic fee: base 1% → max 5% unless user overrides
- tokenAdmin + reward recipient + reward admin: Bankr wallet
- Fee preference: Paired (claim in DIEM)
- originatingChainId: 8453
- context: `{"interface":"liqpad","source":"twitter"}`

## X / social rules

Mentions of @bankrbot from a linked X account use that user's Bankr wallet and this skill.

- Actionable text must be in the first lines.
- Ignore standalone gm/hello.
- Treat attached tweet image as token image unless they give another URL.
- Public reply must be short: name, symbol, CA, pair=DIEM, hook=dynamic, tx, Liquid + Basescan links.
- Always preview and require `yes` / `confirm` before sending the deploy tx, unless they already wrote `confirm` in the same post.
- If confirmation is needed, reply with the preview and wait. Do not deploy on the first vague tweet.

## Command routing

Parse loosely. Examples in `references/commands.md`.

| Intent | Examples | Action |
|---|---|---|
| Deploy | `liqpad deploy Neon NEON`, `liqpad launch Neon NEON pair DIEM` | Preview → confirm → deploy |
| Fees | `liquid fees`, `liquid fees NEON` | Read claimable, do not send tx |
| Claim | `claim liquid`, `claim liquid 0x...` | Show amounts → confirm → claim |
| Status | `liquid status 0x...` | deploymentInfo + fee state |

## Deploy workflow

1. Resolve signer
   - Call Bankr wallet identity (`/wallet/me` or built-in wallet tool).
   - `admin = wallet.address`.
2. Parse name + symbol. Stop if either is missing.
3. Image = tweet image or `image:` URL or empty.
4. Fetch DIEM USD price from
   `https://api.geckoterminal.com/api/v2/simple/networks/base/token_price/0xf4d97f2da56e8c3098f3a8d538db630a2606a024`
   If fetch fails, ask for a DIEM USD price. Do not invent one.
5. Build positions with `createLiquidPositionsUSD(startingCapUsd, diemPriceUsd)`.
   Default `startingCapUsd = 20000` unless user set `mcap:` or `tick:`.
   If they set `tick:`, use that as `tickIfToken0IsLiquid` and still require position arrays (use SDK helper around that tick if available).
6. Preflight on Base:
   - factory `deprecated()` must be false
   - `enabledLockers(LP_LOCKER, HOOK_DYNAMIC_FEE_V2)` must be true
7. Show preview:
   - name / symbol / image
   - pair DIEM
   - hook Dynamic Fee V2
   - admin / fee recipient = Bankr wallet
   - start cap and DIEM price used
   - fee curve
   - "Factory deploys fixed LiquidToken (100B, 18 decimals). LP locks forever."
8. Wait for confirm unless the same message already contains confirm/yes/deploy now.
9. Execute deploy:
   Preferred: `liquid-sdk` `deployToken` with a wallet client that signs through Bankr `/wallet/sign` + `/wallet/submit` (see `scripts/bankr-liquid-deploy.md`).
   Fallback: encode `ILiquid.deployToken(DeploymentConfig)` and submit raw tx:
   - to = factory
   - chainId = 8453
   - value = 0 unless a payable extension is used
   - RPC = `BASE_RPC` or default Base
10. Reply with:
    - token address
    - tx hash
    - https://basescan.org/token/{token}
    - https://app.liquidprotocol.org/tokens/{token}
    - https://basedbot.app/token/base/{token}
    - pair = DIEM, hook = dynamic
    - "Fees accrue automatically. Say liquid fees {symbol}"

## Fee and claim workflow

See `references/fees.md`.

Check:
- `getTokenRewards(token)`
- `getFeesToClaim(wallet, token)`
- `getAvailableFees(wallet, token)`
- `getPoolFeeState(poolId)` if available

Claim:
- Only if wallet is a reward recipient
- `collectRewards(token)` then `claimFees(wallet, token)`
- Confirm if claimable value looks non-trivial
- Return tx hashes + asset amounts (expect DIEM and/or token)

## Hard rules

- Do not deploy on Robinhood / Solana for this skill.
- Do not pair WETH unless the user explicitly overrides `pair:WETH`.
- Do not reuse WETH tick `-230400` silently on a DIEM pair. Say that quote units changed if you ever use that tick.
- Do not run local `liqpad-deploy.ts` that loads `DIEMPLOYER_TEST`.
- Do not change reward recipient unless current admin asks.
- Custom ERC-20 bytecode cannot be injected. LiquidToken is fixed.
- If factory/hook/locker preflight fails, stop and report the revert reason.

## After deploy

Offer:
- `liquid status {token}`
- `liquid fees {token}`
- optional Bankr automation: "claim liquid fees for {token} every 24h" only if user asks
