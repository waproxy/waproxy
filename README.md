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

## WAPROXY-CORE-EXAMPLE-V3

TITLE:
WAproxy Multi-Coin Market Send Examples (USDT→BTC/LTC/DOGE)

VERSION:
V3 (Philosophy Locked)

DESCRIPTION:
This document demonstrates numeric examples of sending
a market-referenced value (e.g., 10 USDT) across supported coins.

Coins covered:
- BTC
- LTC
- DOGE

Protocol fee:
FEE_BPS = 300 (3%)

------------------------------------------------------------
GLOBAL EXECUTION RULES
------------------------------------------------------------

1) amount_sat is defined by application logic.
2) protocol_fee = amount_sat × 0.03
3) receiver_gets = amount_sat - protocol_fee
4) wallet_balance >= amount_sat + safety_margin
5) network_fee is dynamic and external.
6) All calculations are integer-based.
7) Floating point arithmetic is avoided.

------------------------------------------------------------
SECTION 1 — BTC EXAMPLE
------------------------------------------------------------

Market reference (example snapshot):
10 USDT ≈ 0.00015 BTC

Bitcoin atomic unit:
1 BTC = 100,000,000 sat

Conversion:
0.00015 × 100,000,000 = 15,000 sat

Set:
amount_sat = 15,000

Protocol fee:
15,000 × 0.03 = 450 sat

Receiver gets:
15,000 - 450 = 14,550 sat

Safety margin example:
500 sat

Minimum wallet balance:
15,000 + 500 = 15,500 sat

------------------------------------------------------------
SECTION 2 — LTC EXAMPLE
------------------------------------------------------------

Market reference (example snapshot):
1 USDT ≈ 0.018 LTC
10 USDT ≈ 0.18 LTC

Litecoin atomic unit:
1 LTC = 100,000,000 sat

Conversion:
0.18 × 100,000,000 = 18,000,000 sat

Set:
amount_sat = 18,000,000

Protocol fee:
18,000,000 × 0.03 = 540,000 sat

Receiver gets:
18,000,000 - 540,000 = 17,460,000 sat

Safety margin example:
100,000 sat

Minimum wallet balance:
18,000,000 + 100,000 = 18,100,000 sat

------------------------------------------------------------
SECTION 3 — DOGE EXAMPLE
------------------------------------------------------------

Market reference (example snapshot):
1 USDT ≈ 10 DOGE
10 USDT ≈ 100 DOGE

Dogecoin atomic unit:
1 DOGE = 100,000,000 sat (internal WAproxy unit)

Conversion:
100 × 100,000,000 = 10,000,000,000 sat

Set:
amount_sat = 10,000,000,000

Protocol fee:
10,000,000,000 × 0.03 = 300,000,000 sat

Receiver gets:
10,000,000,000 - 300,000,000 = 9,700,000,000 sat

Safety margin example:
100,000,000 sat

Minimum wallet balance:
10,000,000,000 + 100,000,000
= 10,100,000,000 sat
CALCULATION CONTRACT (NON-NEGOTIABLE):

- All amounts are integer SAT units.
- Protocol fee is deducted from amount_sat.
- Network fee is deducted from wallet balance.
- Safety margin prevents execution failure.
- WAproxy never guarantees fiat equivalence.
- 
OPTIONAL REVERSE CALCULATION


If exact net delivery is required:

amount_sat = desired_net / 0.97

Applications must handle rounding safely.

 ## CALCULATION CONTRACT (NON-NEGOTIABLE):

- All amounts are integer SAT units.
- Protocol fee is deducted from amount_sat.
- Network fee is deducted from wallet balance.
- Safety margin prevents execution failure.
- WAproxy never guarantees fiat equivalence.
------------------------------------------------------------
PROTOCOL PHILOSOPHY
------------------------------------------------------------

WAproxy does not price assets.
WAproxy executes deterministic value transport.

------------------------------------------------------------

END OF DOCUMENT
----
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
