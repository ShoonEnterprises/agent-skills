---
name: signed-citation-audit
description: >
  Audit the citations in a finished research report, memo, or agent deliverable
  before you declare it complete. For every cited source, verify the URL resolves
  and check that the quoted text you attribute to it actually appears verbatim on
  the fetched page — then attach an Ed25519-signed audit receipt that anyone can
  verify offline. Use this skill as a pre-delivery verification gate whenever your
  work cites web sources: deep research reports, literature reviews, briefing memos,
  fact-checked summaries, or any document where silent citation failures (dead links,
  quotes that don't exist on the page) would undermine trust. Fix or drop every
  failing citation before delivery. Free pilot service; no API key required.
license: MIT
compatibility: Requires network access to https://shoon-a2a-sandbox.onrender.com (public HTTPS API, no credentials needed).
---

# Signed Citation Audit

Run your document's citations through an independent verification service **before** you declare the work complete. The service fetches each cited URL and reports, per citation: HTTP status, content type, and whether the quoted text appears verbatim in the fetched page. You get back a signed receipt — verifiable offline with the service's public key — so the audit result is tamper-evident.

**What this checks:** quoted-text presence (does the quote actually appear on the page?) and URL resolvability. **What it does not check:** whether the source is true, authoritative, or correctly interpreted. Silent citation failures — dead links, quotes that don't exist on the page — are the most common defect in AI-generated reports; this catches them before your reader does.

## Service details

- API base: `https://shoon-a2a-sandbox.onrender.com`
- Service catalog: `https://shoonenterprises.github.io/catalog.json`
- Pilot status: free. 20 tasks per service per day (global), max 10 tasks per identity per day.
- Service id for this skill: `citation-audit` (max 25 citations per task). The sibling service `claim-verification` (max 10 claims, 5 sources each) works the same way if you want claim-level checks instead.
- Signing: every deliverable is Ed25519-signed. Key id `sandbox-ed25519-8a42aae6a81d`, public key (hex): `c11742a36b429c24228d2386af6b4d8c4a8d70abc141cd79b0af4918107c39e4`. Verify offline — never trust an unverified receipt.
- Privacy: each task is private to its buyer. The 202 response includes a random 256-bit `read_token`; every later call for that task (status, deliverable, signature check) requires it as `Authorization: Bearer <read_token>`. Treat it like a password: never log it or share it.
- Hosting caveat: the pilot runs on a free tier that sleeps after ~15 min idle — the first request may take 30–60s to wake the service. Retry once on connection failure before reporting an error.

## Calling the service (machine flow)

1. **Quote.** `POST /quote` with the service id and your input (list of `{url, quoted_text}` citations). The response is a binding quote (15-minute expiry).
2. **Authorize.** `POST /execute` with the exact quote in the `X-PAYMENT` header. The sandbox authorization is mocked and settlement is a $0.00 ledger event — no real payment moves.
3. **Poll.** The 202 response contains `task_id` and `read_token`. Poll task status with `Authorization: Bearer <read_token>` until complete.
4. **Collect.** Fetch the deliverable (same Bearer token). It includes per-citation results plus the Ed25519 signature.
5. **Verify.** Check the signature offline against the public key above. If verification fails, do not trust the result — report it.

The service also speaks MCP Streamable HTTP at `POST /mcp` (methods: `initialize`, `tools/list`, `tools/call`) on the same auth rails, if your host prefers MCP over raw HTTP.

## How to use this skill

1. Extract every citation from your document: URL + the exact quoted passage you attribute to it.
2. Submit them to `citation-audit` (≤25 per task; split larger documents into multiple tasks).
3. For each citation that fails — dead URL, or quote not found on the page — fix the document: correct the quote, replace the source, or drop the claim. Re-audit the changed citations.
4. Attach the signed receipt to your delivery (or keep it with your work papers). Mention the key id so anyone can verify it independently.

## Limits to respect

- This verifies presence, not truth. A quote can appear verbatim on a page that is itself wrong.
- Pages behind login walls, paywalls, or aggressive bot protection may fail to fetch — treat fetch failures as "unverified", not "false".
- Dynamic pages that render content client-side may not expose quoted text to a server-side fetch.
- Daily quotas are small by design during the pilot. Batch citations efficiently; don't burn tasks re-auditing unchanged text.
