# Execute deploy with Bankr wallet

const creator = bankrWallet;
const liqpad  = "0xa0D2667DD863257B85D0593AED3ee791F48F1B10";

await liquid.deployToken({
  name,
  symbol,
  image,
  tokenAdmin: creator,
  metadata,
  context: JSON.stringify({ interface: "liqpad", source: "twitter" }),
  hook: ADDRESSES.HOOK_DYNAMIC_FEE_V2,
  pairedToken: EXTERNAL.DIEM,
  poolData: encodeDynamicFeePoolData({
    baseFeeBps: 100,
    maxFeeBps: 500,
    referenceTickFilterPeriod: 300,
    resetPeriod: 3600,
    resetTickFilter: 100,
    feeControlNumerator: 500000000n,
    decayFilterBps: 5000,
  }),
  ...positions,
  rewardAdmins:     [liqpad, creator],
  rewardRecipients: [liqpad, creator],
  rewardBps:        [2000, 8000],
});

Sign via POST /wallet/sign (eth_signTransaction)
Submit via POST /wallet/submit, chainId 8453
RPC = BASE_RPC || default Base

Never use DIEMPLOYER_TEST or privateKeyToAccount.
