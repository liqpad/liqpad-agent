# Commands

## Deploy
liqpad deploy <Name> <SYMBOL>
liqpad deploy <Name> <SYMBOL> pair:DIEM fees:dynamic
liqpad deploy <Name> <SYMBOL> mcap:20000 image:https://...
liqpad launch <Name> <SYMBOL>
@bankrbot use liqpad to deploy Neon NEON on base

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
- admin:<address> (default Bankr wallet)
- recipients:<addr:bps>,...  bps must sum to 10000
- confirm | yes | deploy now
