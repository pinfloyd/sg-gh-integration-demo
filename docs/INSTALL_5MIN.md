# Secrets-Guard Enterprise Runtime (v0.1) — 5-minute install

This document describes a minimal self-hosted install that enforces a signed DENY/ALLOW decision in GitHub Actions.

## Fixed anchors (v0.1)

AUTHORITY_URL: http://107.172.34.21:8787

## What this is

A GitHub Actions workflow computes **diff-only (added lines only)** from the push range and calls an external Authority endpoint (/admit). The Authority returns a signed admission record. If decision = DENY, the workflow fails (hard block).

## What this is NOT

No SaaS. No telemetry. No support promises. No multi-tenant.

---

# 0) Prerequisites (Authority host)

- Docker installed and running
- TCP 8787 reachable from GitHub Actions runners
- SSH administration via key-based access

---

# 1) Quick connectivity check

curl -sS http://107.172.34.21:8787/pubkey

Expected: HTTP 200 and JSON containing public_key_sha256.

---

# 2) GitHub Actions integration (real diff-only)

This repo contains `.github/workflows/sg.yml` which:

1) checks out repo with full history (`fetch-depth: 0`)  
2) computes `git diff --unified=0 ${{ github.event.before }} ${{ github.sha }}`  
3) extracts **added lines only** into `diff_facts[]`:
   - path
   - line (new-file line number)
   - added (exact added line text)
4) POSTs intent to:
   http://107.172.34.21:8787/admit
5) reads `.signed_record.decision`
6) exits 1 on DENY (hard fail)

NOTE: the workflow does not print raw diff_facts (to avoid leaking sensitive content into CI logs).

---

# 3) Validate enforcement

Push a commit that adds a secret-like string. Expected:

- Authority returns DENY
- GitHub Actions job FAILS

---

# Troubleshooting

## Connection timeout/refused
- Ensure port 8787 is reachable from the internet
- Ensure service listens on 0.0.0.0:8787 (not only 127.0.0.1)

## Job succeeds while decision=DENY
- Ensure jq reads `.signed_record.decision`
- Ensure `exit 1` is executed on DENY