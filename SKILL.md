---
name: liqpad
description: Deploy Liquid Protocol tokens on Base paired with DIEM using the dynamic-fee hook, split LP creator fees 20% to liqpad treasury 0xa0D2667DD863257B85D0593AED3ee791F48F1B10 and 80% to the token creator, then check and claim those Liquid fees. Use when the user says use liqpad to deploy, liqpad launch, pair with DIEM, dynamic fees, check liquid fees, claim liquid fees, or mentions factory 0x04F1a284168743759BE6554f607a10CEBdB77760. Do NOT use Bankr Doppler/Clanker launch for these phrases.
tags: [liquid, diem, base, tokens, fees, deploy, launch, bankrbot, liqpad]
version: 2
visibility: public
metadata:
  clawdbot:
    emoji: "💧"
    homepage: "https://app.liquidprotocol.org/docs"
    requires:
      packages: [liquid-sdk, viem]
---

# Liqpad — Liquid DIEM Launcher for BankrBot

Launches tokens through **Liquid Protocol on Base**, not Bankr Doppler.

Signer is the **Bankr wallet of the user who mentioned @bankrbot** (the token creator).
RPC is `BASE_RPC` if set, otherwise default public Base RPC.
Never read or ask for a private key. Never use `privateKeyToAccount`.

## When to load

Load this skill if the message contains any of:
- use liqpad to deploy / liqpad deploy / liqpad launch
- pair DIEM / pair $DIEM / paired with diem
- dynamic fees / HOOK_DYNAMIC_FEE
- liquid fees / claim liquid / liquid status
- factory `0x04F1a284168743759BE6554f607a10CEBdB77760`

If the user only says "deploy a token" with no liqpad / Liquid / DIEM words, do **not** use this skill.

## Defaults

See `references/addresses.md`.

- Chain: Base `8453`
- Factory: `0x04F1a284168743759BE6554f607a10CEBdB77760`
- Hook: Dynamic Fee V2
- Locker: LP Locker Fee Conversion
- MEV: Sniper Auction V2
- Quote: DIEM `0xf4d97F2da56e8c3098f3a8D538DB630A2606a024`
- Tick spacing: 200
- Starting cap: $20,000 USD using live DIEM USD price
- Dynamic fee: base 1% → max 5% unless overridden
- tokenAdmin: creatorWallet (caller Bankr wallet)
- LIQPAD_FEE_WALLET: `0xa0D2667DD863257B85D0593AED3ee791F48F1B10`
- rewardRecipients: `[LIQPAD_FEE_WALLET, creatorWallet]`
- rewardAdmins: `[LIQPAD_FEE_WALLET, creatorWallet]`
- rewardBps: `[2000, 8000]`  (20% liqpad / 80% creator)
- feePreference: `[Paired, Paired]` (claim in DIEM, one entry per recipient)
- originatingChainId: 8453
- context: `{"interface":"liqpad","source":"twitter"}`

## Fee split (mandatory on every new deploy)

| Index | Recipient | Role | BPS | Share |
|---|---|---|---|---|
| 0 | `0xa0D2667DD863257B85D0593AED3ee791F48F1B10` | liqpad treasury | 2000 | 20% |
| 1 | caller Bankr wallet | token creator | 8000 | 80% |

`rewardBps` MUST sum to `10000`.
`rewardAdmins`, `rewardRecipients`, `rewardBps`, and `feePreference` MUST be the same length.

Do **not** send 100% to the Bankr wallet.
Do **not** skip the liqpad recipient.
If the user passes `recipients:`, use that only if bps sum to 10000; otherwise keep 20/80.

This split applies to **new deploys only**. Already-launched tokens keep their original recipients.

## X / social rules

- Actionable text in the first lines.
- Ignore standalone gm/hello.
- Tweet image = token image unless `image:` is set.
- Public reply: name, symbol, CA, pair=DIEM, hook=dynamic, fee split 20/80, tx, Liquid + Basescan + Basedbot links.
- Preview and require `yes` / `confirm` unless the same post already contains confirm / yes / deploy now.

## Command routing

See `references/commands.md`.

| Intent | Examples | Action |
|---|---|---|
| Deploy | `liqpad deploy Neon NEON`, `use liqpad to deploy Neon NEON pair DIEM` | Preview including 20/80 split → confirm → deploy |
| Fees | `liquid fees`, `liquid fees NEON` | Read caller's claimable slice only |
| Claim | `claim liquid`, `claim liquid 0x...` | Show amounts → confirm → claim caller's slice |
| Status | `liquid status 0x...` | deploymentInfo + recipients + bps |

## Deploy workflow

1. Resolve signer via Bankr `/wallet/me`.  
   `creatorWallet = wallet.address`  
   `liqpadWallet = 0xa0D2667DD863257B85D0593AED3ee791F48F1B10`
2. Parse name + symbol. Stop if either is missing.
3. Image = tweet image or `image:` URL or empty.
4. Fetch DIEM USD price from  
   `https://api.geckoterminal.com/api/v2/simple/networks/base/token_price/0xf4d97f2da56e8c3098f3a8d538db630a2606a024`  
   If fetch fails, ask for a price. Do not invent one.
5. Positions = `createLiquidPositionsUSD(startingCapUsd, diemPriceUsd)`.  
   Default cap `20000`. Honor `mcap:` or `tick:`.
6. Preflight on Base:
   - `deprecated()` is false
   - `enabledLockers(LP_LOCKER, HOOK_DYNAMIC_FEE_V2)` is true
7. Preview MUST include:
   - name / symbol / image
   - pair DIEM
   - hook Dynamic Fee V2
   - tokenAdmin = creatorWallet
   - fee split:
     - 20% (2000 bps) → `0xa0D2667DD863257B85D0593AED3ee791F48F1B10`
     - 80% (8000 bps) → creatorWallet
   - start cap and DIEM price
   - fee curve
   - "LiquidToken is 100B, 18 decimals. LP locks forever."
8. Wait for confirm unless already present.
9. Execute `deployToken` with the locker arrays above.  
   Sign with Bankr `/wallet/sign` + `/wallet/submit`.  
   See `scripts/bankr-liquid-deploy.md`.
10. Reply with token, tx, explorers, pair=DIEM, hook=dynamic, split 20/80.

## Fee and claim workflow

See `references/fees.md`.

- `getTokenRewards(token)` must show both recipients.
- Caller can only check/claim the 80% slice.
- Liqpad treasury claims the 20% from `0xa0D2667DD863257B85D0593AED3ee791F48F1B10`.
- `collectRewards(token)` then `claimFees(owner, token)` as that owner.

## Hard rules

- Base only. No Robinhood / Solana.
- Pair DIEM unless user explicitly sets `pair:WETH`.
- Do not silently reuse WETH tick `-230400` on a DIEM pair.
- No private keys. No `DIEMPLOYER_TEST`.
- Do not change recipients after deploy unless the matching rewardAdmin asks.
- LiquidToken bytecode is fixed.
- Stop on factory/hook/locker preflight failure.

## After deploy

Offer:
- `liquid status {token}`
- `liquid fees {token}`
