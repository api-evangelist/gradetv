---
name: Pay for chat, NSFW or producer contact with x402
description: Handle Grade's 402 responses — read live pricing, buy a 30-day pass or pay per call in USDC on Base, and retry with X-PAYMENT.
api: openapi/gradetv-openapi.json
operations: [billing, chat_pass, chat_send, nsfw_pass_buy, contact]
generated: '2026-09-05'
method: generated
---

# Paid actions via x402

Grade sells three things, each $0.10 USDC on Base (chain 8453), settled
through the x402 facilitator `https://facilitator.payai.network`.

1. **Read the live numbers first** — `billing` (`GET /api/billing`, no auth)
   returns prices, the pay_to address, asset contract, facilitator URL and the
   library caps. Never hardcode prices.
2. **The 402 loop** — any paid route without payment answers **402** with an
   x402 `accepts[]` array in the body. Pay per the accepts entry, then repeat
   the SAME call with the `X-PAYMENT` header. On MCP, credentials and
   `X-PAYMENT` go in headers and are passed through.
3. **Chat** — `chat_pass` (`POST /api/chat/pass`) buys/confirms a 30-day
   write pass; then `chat_send`
   (`POST /api/chat/{channel_id}/mensagens`) posts into a channel room.
   Reading chat is free.
4. **NSFW** — consent first (`nsfw_consent_accept`, free, declares 18+),
   THEN `nsfw_pass_buy` (`POST /api/nsfw/pass`) for the 30-day pass.
   Consent without the pass still gets 402; the pass without consent is
   refused.
5. **Producer contact** — `contact` (`POST /api/contact`): humans pass a
   Turnstile token (`form_ts`); agents pay $0.10 via x402 or prepaid credit
   (`POST /api/credito` recharges a credit token that any paid route accepts).

Cautions: passes are 30-day, non-refundable via the API — confirm with the
user before spending. Contact posting has an agent backoff: on 429 honor
`Retry-After`.
