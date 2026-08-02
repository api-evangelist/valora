---
name: Discover and trigger Valora shortcuts
description: List the claim/deposit/withdraw/swap-deposit shortcut actions available on a network, trigger one for a wallet, and simulate the returned unsigned transactions before handing them to the user to sign.
api: openapi/valora-api-openapi.yml
operations: [getShortcutsV2, triggerShortcut, simulateTransactions]
generated: '2026-07-21'
method: generated
---

# Discover and trigger Valora shortcuts

Base URL: `https://api.mainnet.valora.xyz`. No authentication. IMPORTANT:
`triggerShortcut` does not execute anything on-chain — it returns **unsigned
transactions** that the wallet owner must sign and submit client-side. Never
present this flow as moving funds by itself.

## Steps

1. **List shortcuts** — `GET /hooks-api/v2/getShortcuts` (`getShortcutsV2`)
   with `networkIds` (and optionally `address`, `appIds`). Each shortcut has
   `id`, `appId`, `name`, `description`, `networkIds`, and a `category` of
   `claim`, `deposit`, `withdraw`, or `swap-deposit`. Do not use the
   deprecated unversioned `/hooks-api/getShortcuts` route.
2. **Trigger one** — `POST /hooks-api/triggerShortcut` (`triggerShortcut`)
   with JSON body `{ address, appId, shortcutId, networkId }` plus the
   shortcut's own trigger inputs: `deposit`/`withdraw` shortcuts require
   `tokens: [{ tokenId, amount, useMax? }]`; `swap-deposit` requires
   `swapFromToken` (tokenId, amount, decimals, isNative, ...). A `400` with
   "No shortcut found" lists the valid shortcut ids for that app.
3. **Simulate before signing** — `POST /simulateTransactions`
   (`simulateTransactions`) with `{ transactions, networkId }` using the
   `data.transactions` array from step 2. Every entry must come back
   `status: "success"`; use `gasNeeded`/`gasPrice` for fee display.
4. **Hand off for signing** — return the unsigned transactions to the user's
   wallet for signature; never fabricate or alter `to`/`data` fields.

## Rules

- Bigint fields (`value`, `gas`, `estimatedGasUse`) are decimal strings.
- There is no idempotency key; triggering twice produces two independent
  unsigned transaction sets (harmless until signed, but avoid confusing the
  user).
- Error envelope reference: `errors/valora-problem-types.yml`.
