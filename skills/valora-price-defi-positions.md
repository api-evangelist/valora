---
name: Price a wallet's DeFi positions with Valora
description: Look up the tokens Valora supports with live USD prices, then fetch and value a wallet address's DeFi positions (app tokens and contract positions) across supported networks.
api: openapi/valora-api-openapi.yml
operations: [getTokensInfoWithPrices, getPositions, getEarnPositions]
generated: '2026-07-21'
method: generated
---

# Price a wallet's DeFi positions with Valora

Base URL: `https://api.mainnet.valora.xyz`. No authentication or API key is
required; responses on hooks routes use a `{ "message": "OK", "data": ... }`
envelope (see `conventions/valora-conventions.yml`).

## Steps

1. **Get the token universe** — `GET /getTokensInfoWithPrices`
   (`getTokensInfoWithPrices`). Returns a map keyed by network-scoped
   `tokenId` (`<networkId>:<address>`) with `symbol`, `decimals`, `priceUsd`,
   and image URLs. Cache this map; every other response references tokens by
   `tokenId`.
2. **Fetch positions** — `GET /hooks-api/getPositions` (`getPositions`) with
   `networkIds` (repeatable: `celo-mainnet`, `ethereum-mainnet`,
   `arbitrum-one`, `op-mainnet`, `base-mainnet`, `polygon-pos-mainnet`) and
   the wallet `address` (0x + 40 hex chars, lowercase). Optionally filter by
   `appIds` (e.g. `aave`, `ubeswap`, `beefy`, `curve`).
3. **Value each position** — for `type: app-token` positions use `supply`,
   `pricePerShare`, and the per-token `priceUsd`/`balance`; for
   `type: contract-position` read `balanceUsd` directly. Token balances and
   bigints are serialized as decimal strings.
4. **(Optional) Earn opportunities** — `GET /hooks-api/getEarnPositions`
   (`getEarnPositions`) with `networkIds` returns pools for the Earn feature
   with `dataProps.yieldRates`, `tvl`, and `safety`.

## Rules

- `400` means invalid parameters; the body is
  `{ "message": "Invalid request", "details": { ...zod tree... } }` — fix the
  request rather than retrying (`errors/valora-problem-types.yml`).
- Reads are safe to retry; there is no rate-limit signaling, so back off on
  errors anyway.
- Positions surface `availableShortcutIds` — hand these to the
  shortcuts skill to act on a position.
