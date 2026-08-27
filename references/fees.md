# Fees

Two streams on Liquid tokens:
1. LP locker rewards (creator share) — collectRewards + claimFees
2. Dynamic LP fee on the pool — volatility between base and max

Bankr Doppler claim APIs will NOT collect these. Use liquid-sdk / locker calls.

Check
- getTokenRewards(token)
- getFeesToClaim(owner, token)
- getAvailableFees(owner, token)
- getPoolConfig(poolId) / getPoolFeeState(poolId)

Claim
- collectRewards(token)
- claimFees(owner, token)
- only current reward recipient

Report amounts in DIEM and token units plus tx hash.
