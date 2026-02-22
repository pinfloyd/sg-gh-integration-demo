# Secrets-Guard Enterprise Runtime (v0.1) — 5-minute install

This document describes a minimal self-hosted install that enforces a signed DENY/ALLOW decision in GitHub Actions.

## Fixed anchors (v0.1)

AUTHORITY_URL: http://107.172.34.21:8787

## What this is

A GitHub Actions workflow calls an external Authority endpoint (/admit). The Authority returns a signed admission record. If decision = DENY, the workflow fails (hard block).

## What this is NOT

No SaaS. No telemetry. No support promises. No multi-tenant.

---

# 0) Prerequisites (server)

On the Authority host (Ubuntu):

- Docker installed and running
- TCP 8787 reachable from GitHub Actions runners
- SSH access for administration (key-based)

---

# 1) Quick connectivity check (from any machine)

Run:

curl -sS http://107.172.34.21:8787/pubkey

Expected: HTTP 200 and JSON containing public_key_sha256.

---

# 2) GitHub Actions integration (copy/paste)

This repo already contains `.github/workflows/sg.yml` configured to call:

http://107.172.34.21:8787/admit

Enforcement logic:

- parses `.signed_record.decision`
- exits 1 on DENY (job FAIL)

---

# 3) Validate enforcement (1 push)

Edit README.md (any change) and push to main.

Expected:

- Authority responds with decision=DENY (for the test payload)
- GitHub Actions job fails

---

# 4) Production wiring (replace test payload)

Replace the static diff_facts payload with real diff-only facts from your scanner/engine (added lines only). Keep:

- POST /admit
- evaluate `.signed_record.decision`
- hard fail on DENY

---

# 5) Offline verification (optional, Enterprise)

Use the published verifier and proof export to validate:

- pubkey_sha256
- signature (ed25519)
- decision_hash
- record_hash
- prev_hash chain
- ledger consistency

(Verifier/proof packaging is anchored separately as Deploy Kit v0.1.)

---

# Troubleshooting

## Connection timeout/refused
- Ensure port 8787 is open in UFW/security group
- Ensure service listens on 0.0.0.0:8787 (not only 127.0.0.1)

## Job succeeds while decision=DENY
- Ensure jq path is `.signed_record.decision`
- Ensure `exit 1` is executed on DENY