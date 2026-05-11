# AI Usage Log for Bug Bounty Research

## Assignment Context

This AI log documents my AI-assisted workflow for the bug bounty research component of INFO5995 Assignment 2. It is separate from the Part A APK reverse-engineering AI log. The work described here focused on security research methodology, source-code review, local proof-of-concept development, evidence organization, and responsible disclosure report drafting.

The primary local evidence directory used during the work was:

```text
D:\bugbounty
```

The requested final AI log location is:

```text
E:\26S1\INFO5995\Assignment2\ai-log
```

## AI Tools Used

- ChatGPT / Codex was used as an interactive research assistant.
- Local PowerShell commands were used to inspect repositories, list files, run local proof-of-concept scripts, and capture outputs.
- GitHub repositories and public bug bounty program rules were reviewed to understand scope and excluded issue classes.
- Burp Suite was used by me for web request inspection in earlier web/API work.
- Local programming environments were used for harmless proof-of-concept validation, including Node.js, PowerShell, Python, Rust/Cargo, and Java/Javac where appropriate.

## Ethical and Scope Controls

The AI assistance was used under the following constraints:

- I focused on assets that were either explicitly in bug bounty scope or locally cloned open-source repositories.
- I avoided destructive testing, denial-of-service testing, social engineering, brute force, and attempts to access third-party user data.
- For blockchain/client repositories, I preferred local source-code review and local unit-test style PoCs rather than testing against production networks.
- For web/API work, I treated unauthenticated boundary checks and owned-account testing as the safe default.
- When a program explicitly warned that a vulnerability class was likely to be duplicated or out of scope, AI assistance was used to deprioritize that path.
- The AI was used to draft reports and evidence manifests, but I reviewed the claims and adjusted severity language to avoid overstating impact.

## High-Level Workflow

### 1. Understanding bug bounty rules and target scope

Prompt summary:

I provided bug bounty scope text and asked the AI to identify which assets, vulnerability classes, and testing methods were allowed.

AI output summary:

The AI extracted program-specific constraints, such as in-scope repositories, non-rewardable issue classes, safe harbor requirements, and excluded submission types. It also identified where local-only validation was preferable.

How I used it:

I used this to avoid testing outside scope and to decide which findings were likely to be accepted, duplicated, or rejected as informational.

### 2. Repository triage and attack-surface mapping

Prompt summary:

I asked the AI to inspect open-source targets such as Safe Chain, Netflix Zuul, Circle Arc Node, Chia GUI, java-tron, Snap Lens API documentation, and Anthropic Claude Code.

AI output summary:

The AI helped identify security-relevant components, including command wrappers, proxy header handling, RPC endpoints, WalletConnect authorization code, daemon TLS configuration, JSON-RPC services, PBFT signature validation, and plugin/hook behavior.

How I used it:

I used the output to choose deeper review areas instead of relying only on automated scanning. The research moved from broad repository inspection into focused code paths where attacker-controlled input reached security-sensitive logic.

### 3. Local proof-of-concept development

Prompt summary:

I asked the AI to create harmless local PoCs and evidence files under `D:\bugbounty`.

AI output summary:

The AI drafted scripts, local test commands, report drafts, attachment manifests, and reproduction steps. The PoCs were designed to demonstrate logic flaws without targeting live services or other users.

How I used it:

I ran or reviewed the PoCs locally, saved outputs, and used those outputs as evidence for report drafting. Where a PoC only modeled behavior rather than fully exercising the application, the report language was adjusted accordingly.

### 4. Report drafting and severity calibration

Prompt summary:

I asked the AI to draft HackerOne/Intigriti style report fields, including title, weakness type, severity, proof-of-concept description, impact, recommended solution, and attachment guidance.

AI output summary:

The AI produced structured English reports with affected components, code evidence, local reproduction steps, expected vs actual behavior, impact, and remediation. It also suggested weakness categories such as CWE-306, CWE-284, CWE-295, CWE-863, and cryptographic or access-control classifications.

How I used it:

I used the reports as drafts, not final unreviewed submissions. I revised claims when impact was uncertain, when an issue was likely a duplicate, or when local evidence was not strong enough for a bounty submission.

## Project-Specific AI Assistance

### Safe Chain

Target:

```text
https://github.com/AikidoSec/safe-chain
```

Main research objective:

Review the package-manager wrapper and process-spawning logic for command injection risk on Windows.

AI-assisted activities:

- The AI helped inspect `safeSpawn.js` and related code paths.
- It identified risky Windows command construction involving escaping, `cmd.exe` behavior, and `shell: true`.
- It generated a harmless local Node.js PoC to demonstrate marker-file creation rather than performing harmful command execution.
- It advised which screenshots and attachments should support a submission, including source-code screenshots, terminal output, PoC script, and generated marker file.
- It drafted a report with the title: `Windows command injection in Safe Chain package-manager wrapper via unsafe cmd.exe argument escaping`.

Evidence produced:

```text
D:\bugbounty\safechain_附件\safe-chain-windows-command-injection-poc.mjs
D:\bugbounty\safechain_附件\safe-chain-poc-output.txt
D:\bugbounty\safechain_附件\safe-chain-poc-injected.txt
```

How I used the AI output:

I used the AI to convert the technical finding into a clear vulnerability report and to identify stronger attachments than a video-only demonstration.

### Netflix Zuul

Target:

```text
https://github.com/Netflix/zuul
```

Main research objective:

Review proxy behavior for issues relevant to Netflix's in-scope open-source target.

AI-assisted activities:

- The AI first reviewed Netflix's open-source scope and out-of-scope issue classes.
- It helped search for proxy-related logic involving forwarded headers, hop-by-hop headers, PROXY protocol behavior, and malformed HTTP parsing edge cases.
- It generated multiple local PoC/evidence directories for different hypotheses.
- When the first two findings were identified as duplicates, the AI helped pivot to deeper or less obvious ideas rather than continuing duplicate-prone submissions.

Evidence produced:

```text
D:\bugbounty\zuul_附件
D:\bugbounty\zuul_附件2
D:\bugbounty\zuul_附件3
D:\bugbounty\zuul_附件4
D:\bugbounty\zuul_附件5
```

How I used the AI output:

I used the outputs as local learning evidence and draft material. I also used AI feedback to avoid submitting findings likely to be duplicated or insufficiently impactful.

### Circle Arc Node

Target:

```text
https://github.com/circlefin/arc-node
```

Main research objective:

Find higher-impact issues in a blockchain node implementation using source review and local validation.

AI-assisted activities:

- The AI helped inspect consensus RPC routing, execution RPC defaults, Quake deployment defaults, txpool behavior, and EVM/block execution policy enforcement.
- It helped install and configure the local Rust/Cargo workflow under non-system paths where possible.
- It created local evidence directories and report drafts for multiple findings.
- It helped distinguish between medium misconfiguration/access-control issues and stronger consensus-policy bypass issues.

Key findings drafted with AI assistance:

- Unauthenticated consensus RPC peer-management endpoints allowed runtime modification of persistent peers when RPC was reachable.
- Quake-generated deployments could expose sensitive EVM JSON-RPC namespaces and pending transaction data by default.
- The addresses denylist appeared to be enforced in txpool admission but not during EVM/block execution, creating a policy bypass if transactions were directly included in execution payloads.

Evidence produced:

```text
D:\bugbounty\arc-node_poc1
D:\bugbounty\arc-node_poc2_remote_signer
D:\bugbounty\arc-node_poc3_rpc_exposure
D:\bugbounty\arc-node_poc4_denylist_execution_bypass
```

How I used the AI output:

I used the AI to structure the evidence chain: source locations, local test outputs, reproduction commands, report drafts, and attachment manifests. I also used it to calibrate severity and avoid claiming network-wide compromise where the local evidence supported only a narrower policy-bypass or exposure claim.

### Chia Blockchain GUI

Target:

```text
https://github.com/Chia-Network/chia-blockchain-gui
```

Main research objective:

Review Electron/GUI, WalletConnect, daemon communication, and TLS trust boundaries.

AI-assisted activities:

- The AI helped inspect WalletConnect session proposal and session request handling.
- It identified a missing method-level namespace enforcement issue, where a connected dApp could request methods not approved in the session namespace.
- It helped write a harmless model PoC showing expected namespace rejection vs actual modeled dispatch.
- It also helped inspect daemon WebSocket TLS configuration and identify `rejectUnauthorized: false` in multiple daemon client paths.
- It generated a local TLS PoC showing that an untrusted self-signed server is accepted when certificate verification is disabled, while a secure control rejects it.

Evidence produced:

```text
D:\bugbounty\chia_gui_poc1_walletconnect_namespace_bypass
D:\bugbounty\chia_gui_poc2_daemon_tls_no_server_verification
```

How I used the AI output:

I used the AI's report drafts to separate two different vulnerability classes: WalletConnect authorization boundary bypass and daemon TLS certificate validation failure. I also used AI guidance to avoid claiming direct fund theft for the WalletConnect issue because additional signing confirmation flows still existed.

### TRON java-tron

Target:

```text
https://github.com/tronprotocol/java-tron
```

Main research objective:

Find high or critical issues affecting protocol, API, TVM, JSON-RPC, or consensus validation.

AI-assisted activities:

- The AI reviewed TRON's program rules and scope, especially the focus on protocol integrity, TVM, APIs, and consensus logic.
- It helped inspect JSON-RPC transaction building, Lite FullNode restrictions, TVM precompiles, and PBFT signature validation.
- It produced several local PoCs and report drafts, ranging from low/medium API integrity issues to potentially higher-impact cryptographic/signature uniqueness issues.

Key findings drafted with AI assistance:

- JSON-RPC `buildTransaction` could compute malformed smart-contract transactions due to unchecked `gas * energyFee` Java `long` overflow.
- Lite FullNode historical-query restrictions were enforced for REST/gRPC but not JSON-RPC.
- TVM `ValidateMultiSign` could count duplicate recovered addresses as separate signer weight when ECDSA malleability produced different signatures for the same signer.
- PBFT commit-data validation could count duplicate signature byte strings from the same SR address instead of enforcing unique recovered SR signer addresses.

Evidence produced:

```text
D:\bugbounty\java_tron_poc1_jsonrpc_fee_overflow
D:\bugbounty\java_tron_poc2_tvm_log_unknown_result
D:\bugbounty\java_tron_poc3_jsonrpc_litefn_filter_bypass
D:\bugbounty\java_tron_poc4_validatemultisign_duplicate_weight
D:\bugbounty\java_tron_poc5_pbft_duplicate_sr_signatures
D:\bugbounty\tron_sig_poc_venv
```

How I used the AI output:

I used the AI to reason through Java source flows, generate small standalone PoCs, and avoid overclaiming. For example, the JSON-RPC overflow was framed as transaction-builder correctness and API integrity rather than an on-chain validation bypass. The signature findings were framed around uniqueness-of-signer validation rather than claiming one key could always forge a full mainnet quorum.

### Snap Lens / Snapchat API Review

Target:

```text
https://lensstudio.snapchat.com/api/
```

Main research objective:

Identify potentially high-impact API issues while following Snapchat's bug bounty rules.

AI-assisted activities:

- The AI reviewed the Snapchat policy text and extracted constraints such as use of test accounts, no social engineering, no bulk scanning, and no third-party data access.
- It helped plan safe API test areas, including product feed SSRF, product feed credential exposure, Ads API IDOR, Public Profile authorized-data boundaries, and OAuth redirect validation.
- It produced safe unauthenticated boundary checks and an authenticated-check script template intended only for my own Snap test account and objects.

Evidence produced:

```text
D:\bugbounty\snap_lens_api_review\00_scope_and_rules.txt
D:\bugbounty\snap_lens_api_review\01_unauth_boundary_checks.txt
D:\bugbounty\snap_lens_api_review\02_public_profile_public_endpoint_checks.txt
D:\bugbounty\snap_lens_api_review\03_ads_gallery_public_checks.txt
D:\bugbounty\snap_lens_api_review\04_high_impact_test_plan_zh.md
D:\bugbounty\snap_lens_api_review\authenticated_results\snap_authenticated_checks_20260507_122614.txt
```

How I used the AI output:

I used it mainly as a test-planning and boundary-checking assistant. I did not use it to run aggressive automated scanning or to access data belonging to others.

### Anthropic Claude Code

Target:

```text
https://github.com/anthropics/claude-code
```

Main research objective:

Review the open-source repository and npm package for higher-severity issues while avoiding areas likely to be treated as duplicates.

AI-assisted activities:

- The AI reviewed Anthropic's bug bounty rules and highlighted that command-execution permission modal bypasses were likely part of an ongoing remediation project and therefore duplicate-prone.
- It inspected the public repository structure, GitHub Actions, official plugins, npm wrapper package, installer scripts, and Windows native package metadata.
- It identified a `ralph-loop` argument-injection style issue but warned that it was likely within the duplicate-prone command permission bypass class.
- It performed a non-duplicate-oriented review of GitHub Actions permissions, npm wrapper behavior, installer integrity, plugin hook behavior, and security-guidance hook state handling.

How I used the AI output:

I used the AI to decide not to submit a likely duplicate finding. The final conclusion was that, after excluding command-permission-bypass style issues, there was no strong non-duplicate high/critical vulnerability evidence from the public repository alone.

## Validation Performed

Across the research work, the following validation patterns were used:

- Source-code inspection with file and function references.
- Local proof-of-concept scripts that modeled or exercised the vulnerable logic.
- Local unit-test style validation for Rust and Java projects where feasible.
- Saved terminal output showing PoC execution results.
- Attachment manifests listing which files should be submitted as evidence.
- Report drafts separating summary, steps to reproduce, impact, recommended remediation, and suggested weakness classification.

Examples of generated validation artifacts include:

```text
*_poc_output.txt
*_report_draft.md
*_source_evidence.txt
*_attachment_manifest.md
*_submission_fields.md
```

## Human Review and Corrections

The AI was not treated as authoritative. I reviewed and corrected its outputs in several ways:

- I asked for Chinese explanations when I needed to understand the risk before drafting English reports.
- I asked the AI to lower or qualify severity when a bug was only locally modeled or depended on a specific deployment configuration.
- I stopped pursuing findings when they appeared likely to be duplicates, such as the first Zuul findings and the Anthropic command-permission bypass class.
- I asked for stronger evidence chains rather than relying only on videos.
- I separated local learning PoCs from actual submissions and avoided claiming real-world exploitation without sufficient proof.
- I preferred one vulnerability per report, consistent with program rules.

## Limitations

- Some PoCs were local models rather than full end-to-end exploitation against a running production-equivalent deployment.
- Some repositories contained only partial source code or wrappers, so deeper findings would require binary reverse engineering or access to non-public implementation details.
- Some test plans were not executed against live services because doing so could risk violating program rules or accessing third-party data.
- Severity recommendations were preliminary and would need final validation by the bug bounty program triage team.
- AI-generated drafts may contain mistakes, so every report requires manual verification before submission.

## Deliverables Produced with AI Assistance

The AI helped produce:

- Local PoC scripts.
- Static source evidence files.
- Terminal output logs.
- HackerOne/Intigriti style report drafts.
- Attachment checklists.
- Screenshot naming guidance.
- Submission field suggestions.
- Non-submission rationale for duplicate-prone or weak findings.

## Reflection on AI Use

The most useful role of AI was not simply generating vulnerability ideas. Its main value was in structuring the research process: narrowing scope, finding relevant code paths, creating safe local validation plans, drafting clear reports, and checking whether a claim was too broad for the evidence.

The main risk was overconfidence. Some AI-suggested paths initially sounded severe but became weaker after checking exploitability, deployment assumptions, duplicate risk, or program exclusions. For that reason, I treated AI output as a starting point for manual review rather than as final proof.

Overall, AI assistance improved efficiency and documentation quality, but the final responsibility for scope compliance, evidence quality, and report accuracy remained with me.
