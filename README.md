# Waproxy — Wallet Proxy Protocol

Waproxy is a deterministic, curl-first Wallet Proxy Protocol for Bitcoin (BTC), Litecoin (LTC), and Dogecoin (DOGE).

It provides a minimal HTTP JSON surface for multi-chain wallet operations with built-in idempotency and a lightweight proxy layer architecture.

---

## Why Waproxy?

Waproxy is designed for:

- CLI-first execution (no SDK required)
- Deterministic API behavior
- Idempotent transaction handling
- Stateless gateway architecture
- Multi-chain uniformity (BTC / LTC / DOGE)

Infrastructure for engineers — not a consumer wallet UI.

---

## Architecture

Client (curl / automation agent)
→ WAproxy API layer
→ Chain adapter (BTC / LTC / DOGE)
→ Node / RPC backend
→ Blockchain network

Waproxy acts as a proxy layer that:

- Normalizes chain-specific logic
- Shields backend nodes
- Enforces idempotency
- Keeps surface area minimal

---

## Quick Start (BTC Example)

BASE="http://api.waproxy.org"

# 1. Create API key
KEY=$(
  curl -sS -X POST "$BASE/v1/btc?mode=key_create" \
    -H "Content-Type: application/json" \
  | jq -r '.result.key'
)

# 2. Create wallet
WID=$(
  curl -sS -X POST "$BASE/v1/btc?mode=wallet_create" \
    -H "Content-Type: application/json" \
    -H "X-WAPROXY-KEY: $KEY" \
    --data-raw '{"net":"main"}' \
  | jq -r '.result.wallet_id'
)

# 3. Generate receive address
ADDR_NEW=$(
  curl -sS -X POST "$BASE/v1/btc?mode=wallet_receive" \
    -H "Content-Type: application/json" \
    -H "X-WAPROXY-KEY: $KEY" \
    -d "{\"wallet_id\":\"$WID\"}" \
  | jq -r '.result.address'
)

# 4. Send transaction (idempotent)
curl -sS -X POST "$BASE/v1/btc?mode=wallet_send" \
  -H "Content-Type: application/json" \
  -H "X-WAPROXY-KEY: $KEY" \
  -H "X-WAPROXY-IDEMPOTENCY: btc-send-001" \
  -d "{
    \"wallet_id\":\"$WID\",
    \"to\":\"bc1xxxxxxxx\",
    \"amount_sat\":10000
  }" | jq .

---

## Idempotency Model

Header:

X-WAPROXY-IDEMPOTENCY: <unique-id>

If a request is retried with the same idempotency value, the original operation result is returned instead of broadcasting a duplicate transaction.

Prevents double spending caused by network retries or client crashes.

---

## Supported Chains

- Bitcoin (BTC)
- Litecoin (LTC)
- Dogecoin (DOGE)

Chain routing pattern:

/v1/btc
/v1/ltc
/v1/doge

---

## API Pattern

All operations follow:

POST /v1/{chain}?mode={operation}

Examples:

- mode=key_create
- mode=wallet_create
- mode=wallet_receive
- mode=wallet_balance
- mode=wallet_send

---

## Core Design Principles

- Deterministic execution
- Minimal API surface
- Multi-chain abstraction
- CLI-first infrastructure
- Proxy-based wallet architecture

---

## Use Cases

- Stateless payment gateways
- Machine-to-machine payments
- Automation agents
- Server-side crypto infrastructure
- CLI-based payment flows

---

## Security Notes

- Never expose node RPC directly to the internet
- Always use HTTPS in production
- Rotate API keys
- Enforce idempotency on side-effect operations
- Validate address formats per chain

---

## Documentation

Website: https://waproxy.org
LLM Spec: https://waproxy.org/llms-full.txt
RSS: https://waproxy.org/rss.xml
Sitemap: https://waproxy.org/sitemap.xml

---

## Keywords

wallet proxy protocol
wallet proxy architecture
curl-first api
deterministic payment api
idempotent transaction api
multi-chain wallet api
stateless payment gateway

---

## License

Open protocol. Implementation license to be defined by project maintainers.
