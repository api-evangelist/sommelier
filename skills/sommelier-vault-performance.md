---
name: Track Sommelier vault performance
description: >-
  Retrieve APY, TVL and share-price history for a Sommelier (Somm) vault, plus
  protocol-wide total value locked, using the public read-only Sommelier Finance
  API. No authentication is required.
api: openapi/sommelier-api-openapi.yml
operations: [getTvl, getSnapshot, getDailyData, getHourlyData, getAllCellarsDailyData]
---

# Track Sommelier vault performance

The Sommelier Finance API is public and read-only. There is no API key or OAuth —
send plain HTTP GET requests to `https://api.sommelier.finance`. Every response
is a JSON object with a single top-level `Response` field.

## Conventions to follow
- All inputs are **path segments**, not query parameters: network, vault/cellar
  address (Ethereum `0x...`), unix timestamps, block number.
- The only supported `network` value is `ethereum`.
- For the end timestamp you may pass the literal `latest` to fetch all data up
  to now.
- APY values are decimal-adjusted (`13.5` means 13.5%).
- No pagination — bound results with the start/end unix-timestamp segments.
- No documented error envelope or rate limits; add your own retry/backoff.

## Steps

1. **Protocol overview** — call `getTvl` (`GET /tvl`) for total value locked
   across all cellars.
2. **Pick a vault** — identify the cellar's Ethereum address.
3. **Current cellar state** — call `getSnapshot`
   (`GET /snapshot/ethereum/{cellarAddress}`) for USD TVL, base-asset TVL, SOMM
   price, SOMM-per-day emissions and SOMM incentive APY.
4. **Daily history** — call `getDailyData`
   (`GET /dailyData/ethereum/{vaultAddress}/{start}/latest`) and read
   `daily_apy`, `share_price`, `tvl` from each snapshot in `Response`.
5. **Finer resolution** — for intraday movement call `getHourlyData`
   (`GET /hourlyData/ethereum/{vaultAddress}/{start}/{end}`); the hourly
   response omits APY.
6. **Compare across vaults** — call `getAllCellarsDailyData`
   (`GET /dailyData/ethereum/allCellars/{start}/{end}`); the `Response` is keyed
   by cellar address.
