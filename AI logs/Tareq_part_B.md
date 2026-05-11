# Part B Research Summary

## Overview

We reviewed multiple repositories under:

`/Users/tariqalsubaie/Documents/Uni Work/Intro To Cybersec`

The goal was to identify bugs or security issues and shape the stronger findings into HackerOne-style reports. The review covered documentation, OAuth flows, infrastructure tools, web applications, plugins, Mapbox-related repositories, Sui CCTP, and Cosmos EVM.  [oai_citation:0‡Tareq_part_B.md](sediment://file_000000004f9871faa675c4326ef44417)

---

## Docs / OAuth Review

In the earlier authentication documentation review, the following issues were identified:

- **P1:** OAuth secrets in URL query strings.
- **P1:** OAuth flow missing `state` and PKCE guidance.
- **P2:** Insecure HTTP redirect URI example.
- **P2:** Deprecated OAuth draft / legacy user-agent flow guidance.
- **P2:** Documentation normalising password-based Basic Auth.
- **P3:** Mixed HTTP links.
- **P3:** Invalid JSON sample.

**Clarification:**  
The P1/P2/P3 labels were used as review priority labels. They do not always represent the final HackerOne severity.

---

## test-infra / GCSWeb

We reviewed `test-infra/gcsweb` and found the following issues:

- **P2:** OAuth bearer token is passed as `option.WithAPIKey`, which treats it like a Google API key.
- **P2:** Arbitrary HTML artifacts can be served inline, creating an XSS risk under the GCSWeb origin.
- **P3:** HTTP server has no read, write, or header timeouts.

We drafted reporting guidance for the OAuth/API-key issue. However, because it could not be reproduced, it was treated as **Medium** rather than **High** or **Critical**.

---

## Matomo

The Matomo review mostly identified hardening issues:

- **P3:** Vulnerable locked JavaScript dependencies in `package-lock.json`.
- **P3:** CSP allows `unsafe-inline` and `unsafe-eval`.

**Conclusion:**  
No strong High/Critical bounty-grade finding was confirmed unless a clearer exploit path can be proven.

---

## plugin-Slack

We reviewed:

`/Users/tariqalsubaie/Documents/Uni Work/Intro To Cybersec/plugin-Slack`

The following issues were identified:

- **P2:** Global Slack OAuth token can post to arbitrary user-supplied channel IDs.
- **P3:** Slack OAuth token is sent as a request body field instead of using the `Authorization` header.
- **P3:** Slack alert messages interpolate user-controlled names into `mrkdwn` without escaping.

**Best finding:**  
The strongest issue in this repository was the global-token/channel-boundary issue.

---

## Mapbox

### mapbox-gl-native

In `mapbox-gl-native`, the following issues were found:

- **P2:** TileJSON `maxzoom` up to `255` can trigger offline download DoS or unsafe tile math.
- **P2:** Remote resources are buffered into memory without size limits.
- **P3:** `asset://` paths can escape the configured asset root.

### delaunator

In `delaunator`, the following issue was found:

- **P3:** Invalid coordinates after the first value are accepted, which can cause corrupted triangulation or possible DoS.

### mapbox-gl-js

In `mapbox-gl-js`, the following issues were found:

- **P2:** glTF external subresources bypass `transformRequest`.
- **P2:** Core request helper reads remote responses fully into memory without size limits.
- **P3:** Worker URL is interpolated into Blob module code.

We wrote HackerOne-style material for the first two `mapbox-gl-js` issues.

### glTF External Subresource Issue

For the glTF issue, the selected classification was:

- **Severity:** Medium
- **Weakness:** CWE-284
- **Alternative Weakness:** CWE-602
- **Asset:** GitHub source code repository

### Response-Size DoS Issue

For the response-size DoS issue, the impact was drafted around attacker-controlled map resources causing browser memory exhaustion.

---

## sui-cctp

In `sui-cctp`, the following issues were found:

- **P3:** Deployment helper logs and writes deployer private keys.
- **P3:** Sui token unlink event can report the wrong local token.
- **P3:** EVM unlink event accepts arbitrary `localToken`.

The main focus was the deployer private key issue. It was recommended as **Low/P3 severity**, unless the key is reused with real deployment keys or exposed through CI.

**Note:**  
There was also a brief mistaken check inside `sui-cctp/evm-cctp-contracts/docs`, where vulnerable `ethers` and `web3` dependencies appeared. This was later clarified because the intended target was the separate `evm` folder.

---

## cosmos/evm

We then reviewed the correct repository:

`/Users/tariqalsubaie/Documents/Uni Work/Intro To Cybersec/evm`

### Repository Information

- **Remote:** `https://github.com/cosmos/evm.git`
- **Version:** `v0.7.0-beta.0-26-g6edbe09c`

### Key Findings

- **P1:** Public JSON-RPC can sign with node keyring keys.
- **P1:** `eth_sendTransaction` signs and broadcasts from local keys by default.
- **P2:** Private RPC namespace flag is ignored by HTTP registration.

HackerOne-style reports were written for the first two findings.

---

## Finding 1: Public JSON-RPC Can Sign with Node Keyring Keys

### Recommended Classification

- **Weakness:** CWE-306 — Missing Authentication for Critical Function
- **Alternative Weakness:** CWE-287 — Improper Authentication
- **Severity:** High

### Severity Note

This should only be treated as **Critical** if there is proof of:

- Remotely reachable RPC,
- A funded or privileged local key,
- And direct asset loss or privileged action.

---

## Finding 2: `eth_sendTransaction` Signs and Broadcasts from Local Keys by Default

### Recommended Classification

- **Weakness:** CWE-306 or CWE-862
- **Severity:** High

### Severity Note

This should only be treated as **Critical** if there is a working remote exploit against a funded or privileged node.

---

## Verification Note

An attempt was made to run Go tests for the `evm` repository. However, Go was not installed in the environment, so test execution was not completed.