# DeploymentConfig

TokenConfig
- tokenAdmin: Bankr wallet
- name, symbol, image, metadata, context
- salt: keccak256(name + symbol + wallet + timestamp) if user did not supply one
- originatingChainId: 8453

PoolConfig
- hook: Dynamic Fee V2
- pairedToken: DIEM
- tickIfToken0IsLiquid + tickSpacing 200 from createLiquidPositionsUSD
- poolData: encodeDynamicFeePoolData

Default poolData
- baseFeeBps: 100
- maxFeeBps: 500
- referenceTickFilterPeriod: 300
- resetPeriod: 3600
- resetTickFilter: 100
- feeControlNumerator: 500000000n
- decayFilterBps: 5000

LockerConfig
- locker: LP Locker Fee Conversion
- rewardAdmins / rewardRecipients: [Bankr wallet] unless overridden
- rewardBps: [10000] or user split
- tickLower / tickUpper / positionBps from SDK helper; positionBps sum 10000
- lockerData: abi.encode({ feePreference: [Paired] })
  FeeIn: 0=Both, 1=Paired, 2=Liquid

MevModuleConfig
- mevModule: Sniper Auction V2
- mevModuleData: SDK default encodeSniperAuctionData (80% → 40% over 20s)
  If encoding is unknown, pass the SDK default. Do not pass empty bytes unless SDK says empty is valid.

extensionConfigs: [] unless user asked for vault/airdrop/dev-buy.
