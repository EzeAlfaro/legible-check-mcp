# legible-check — pay-per-call data tools for AI agents (x402 + MCP)

[![Legible Check MCP server](https://mcprush.com/badge/legible-check-mcp/card.svg)](https://mcprush.com/to/legible-check-mcp?ref=badge)
[![Powered by RustChain](https://img.shields.io/badge/Powered%20by-RustChain-orange)](https://rustchain.org)


An MCP server and x402 seller that exposes on-chain data tools an agent can call and pay for
per request in USDC on Base. No signup, no API key, no subscription: an unpaid call answers
HTTP 402 with the price, the payer signs a USDC authorization, and the tool runs.

- **MCP endpoint (Streamable HTTP):** `https://legible-check.ezealfa.workers.dev/mcp`
- **OpenAPI:** `https://legible-check.ezealfa.workers.dev/openapi.json`
- **Health:** `https://legible-check.ezealfa.workers.dev/health`
- **Receiving wallet:** `0xF3D5de46f73d9f55DFce23F4AAF3Db5505c823c5` (Base)

## Connect from an MCP client

```json
{
  "mcpServers": {
    "legible-check": {
      "type": "streamable-http",
      "url": "https://legible-check.ezealfa.workers.dev/mcp"
    }
  }
}
```

The server answers JSON-RPC 2.0 over `POST /mcp`, and also in `text/event-stream` frames when
the client sends `Accept: text/event-stream`.

Methods: `initialize`, `tools/list`, `tools/call`.

## Tools

| tool | price (USDC) | what it returns |
|---|---|---|
| `evm_block` | 0.001 | block number, timestamp, base fee, gas used, tx count |
| `evm_gas` | 0.001 | gas price split into base and priority fee |
| `evm_balance` | 0.002 | native balance in wei and native units |
| `evm_token_balance` | 0.003 | ERC20 balance with decimals and symbol |
| `evm_nonce` | 0.002 | next transaction nonce |
| `evm_tx` | 0.002 | decoded transaction by hash |
| `evm_tx_status` | 0.001 | success / reverted / pending with confirmations |
| `evm_receipt` | 0.002 | receipt: status, block, gas used, effective gas price |
| `evm_transfers` | 0.003 | ERC20 transfers for an address over N blocks |
| `evm_token_info` | 0.002 | name, symbol, decimals, total supply |
| `evm_contract` | 0.003 | is-contract check, bytecode size, Sourcify ABI |
| `evm_portfolio` | 0.005 | native balance + nonce + up to 10 token balances in one call |
| `evm_decode` | 0.003 | calldata decoder: selector, signatures, decoded arguments |
| `crypto_price` | 0.001 | live prices with 24h change and market cap |
| `dex_quote` | 0.002 | DEX pair quotes with liquidity and 24h volume |
| `arena_commit_reveal_roll` | 0.010 | cryptographic commit-reveal randomness benchmark verifying SHA-256 deterministic PRNG |
| `security_ai_vault_ctf` | 0.050 | AI red-teaming security challenge (CTF / Bug Bounty): adversarial prompt injection test |

Chains: `base`, `polygon`, `arbitrum` (pass `?chain=` on HTTP, or `chain` in the tool arguments).

## How payment works

1. Call a tool without payment. The server answers `402` with `accepts[]`: `payTo`, `amount`
   (atomic USDC, 6 decimals), `asset`, `network` (`eip155:8453`).
2. Sign an EIP-3009 `TransferWithAuthorization` for that amount.
3. Retry the same request with the `PAYMENT-SIGNATURE` header containing the base64 payload
   `{ x402Version: 2, accepted: <requirement>, payload: { type: "eip3009", signature, authorization } }`.
4. The server verifies and settles through the CDP facilitator and returns the data plus a
   `PAYMENT-RESPONSE` header with the settlement transaction.

Parameter names are query parameters on HTTP (`?address=`, `?token=`, `?hash=`, `?chain=`).
Unsupported chains are rejected with `400` before any payment is requested.

## Why this exists

The agent economy pays per call, but most endpoints are undocumented wrappers with no schema.
This server publishes a machine-readable contract: OpenAPI with `x-payment-info`, an MCP tool
list with input schemas, and answers that always state the chain and the block they were read at.

## Research & Security Compliance

All tools, benchmarks, and challenges provided by this server are strictly for autonomous AI agent research, cryptographic protocol evaluation, and cybersecurity red-teaming:
- `arena_commit_reveal_roll` demonstrates deterministic SHA-256 pseudo-random commit-reveal verification.
- `security_ai_vault_ctf` operates under an open vulnerability research model (Capture The Flag / Bug Bounty). Fees represent inference computation costs and the prize pool acts as an alignment bounty.
This server does not operate unlicensed gambling, wagering, or custodial financial services.

## License

MIT
