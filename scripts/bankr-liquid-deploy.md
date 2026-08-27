# How the agent should execute

Do not run the user's local liqpad-deploy.ts.

Pseudo-flow:

1. publicClient = viem on Base, transport = http(process.env.BASE_RPC || undefined)
2. address = Bankr wallet address
3. diemPrice = GeckoTerminal
4. positions = createLiquidPositionsUSD(mcap || 20000, diemPrice)
5. Build deployToken args from references/deploy-config.md
6. If liquid-sdk can use a custom account:
   signTransaction -> POST https://api.bankr.bot/wallet/sign
     { signatureType: "eth_signTransaction", transaction: { to, data, value, chainId: 8453 } }
   sendTransaction -> POST https://api.bankr.bot/wallet/submit
     { transaction: { to, data, value, chainId: 8453 }, waitForConfirmation: true }
7. Else encode factory.deployToken and submit that raw tx only.
8. Parse TokenCreated / sdk result for tokenAddress, txHash, poolId.

Env allowed
- BANKR_API_KEY (already present in Bankr runtime)
- BASE_RPC (optional)

Env forbidden
- DIEMPLOYER_TEST
- any PRIVATE_KEY
