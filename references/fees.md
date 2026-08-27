# Fees

LP locker creator rewards are split at deploy:

- 20% (2000 bps) → 0xa0D2667DD863257B85D0593AED3ee791F48F1B10
- 80% (8000 bps) → token creator Bankr wallet

Each recipient claims their own slice.

Check
- getTokenRewards(token)
  expect recipients [0xa0D2667DD863257B85D0593AED3ee791F48F1B10, creator]
  expect bps [2000, 8000]
- getFeesToClaim(owner, token)
- getAvailableFees(owner, token)
- getPoolConfig(poolId) / getPoolFeeState(poolId)

Claim
- collectRewards(token)
- claimFees(owner, token)
- only the matching recipient can claim that slice

When the caller asks "liquid fees" or "claim liquid", use the caller wallet (80%).
Do not claim the liqpad 20% unless the active Bankr wallet IS 0xa0D2667DD863257B85D0593AED3ee791F48F1B10.

Bankr Doppler claim APIs will not collect these fees.
