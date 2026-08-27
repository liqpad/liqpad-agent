# Commands

## Deploy
liqpad deploy <Name> <SYMBOL>
liqpad deploy <Name> <SYMBOL> pair:DIEM fees:dynamic
liqpad deploy <Name> <SYMBOL> mcap:20000 image:https://...
liqpad launch <Name> <SYMBOL>
@bankrbot use liqpad to deploy Neon NEON on base pair DIEM dynamic fees confirm

Default fee split on every deploy:
- 20% 0xa0D2667DD863257B85D0593AED3ee791F48F1B10
- 80% caller Bankr wallet

## Fees
liquid fees
liquid fees <SYMBOL|0xToken>
how much DIEM fees has <SYMBOL> earned

## Claim
claim liquid
claim liquid <SYMBOL|0xToken>

## Status
liquid status <SYMBOL|0xToken>

Optional flags
- pair:DIEM (default)
- fees:dynamic (default) | fees:static
- mcap:<usd>
- tick:<int24>
- image:<url>
- admin:<address> (tokenAdmin only; does not remove the 20% liqpad slice)
- recipients:<addr:bps>,...  bps must sum to 10000
- confirm | yes | deploy now
