# DeploymentConfig

## TokenConfig
- tokenAdmin: creatorWallet (caller Bankr wallet)
- name, symbol, image, metadata, context
- salt: keccak256(name + symbol + wallet + timestamp) if not supplied
- originatingChainId: 8453
- context: {"interface":"liqpad","source":"twitter"}

## PoolConfig
- hook: Dynamic Fee V2
- pairedToken: DIEM
- tickIfToken0IsLiquid + tickSpacing 200 from createLiquidPositionsUSD
- poolData: encodeDynamicFeePoolData

### Default poolData
- baseFeeBps: 100
- maxFeeBps: 500
- referenceTickFilterPeriod: 300
- resetPeriod: 3600
- resetTickFilter: 100
- feeControlNumerator: 500000000n
- decayFilterBps: 5000

## LockerConfig
- locker: LP Locker Fee Conversion
- creatorWallet: Bankr wallet of the user who asked to deploy
- liqpadWallet: 0xa0D2667DD863257B85D0593AED3ee791F48F1B10

- rewardAdmins:     [liqpadWallet, creatorWallet]
- rewardRecipients: [liqpadWallet, creatorWallet]
- rewardBps:        [2000, 8000]

- tickLower / tickUpper / positionBps from SDK helper; positionBps sum 10000
- lockerData: abi.encode({ feePreference: [Paired, Paired] })
  FeeIn: 0=Both, 1=Paired, 2=Liquid
  feePreference length MUST equal rewardRecipients length

## MevModuleConfig
- mevModule: Sniper Auction V2
- mevModuleData: SDK default encodeSniperAuctionData (80% → 40% over 20s)

## extensionConfigs
- [] unless user asked for vault/airdrop/dev-buy
