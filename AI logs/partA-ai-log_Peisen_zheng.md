# INFO5995 Assignment 2 Part A - AI Usage Log

## Scope
This log covers AI-assisted work for the provided APK `a2_case1.apk` only. It does not cover Part B in-the-wild bug bounty work.

## Tools and Materials
- `jadx` cache and local decompiled Java source for static analysis
- `apktool`/unpacked APK directory for manifest/resource inspection
- Local shell commands for text search and evidence extraction
- AI assistance for requirement interpretation, report structuring, mitigation checking, and mock Q&A rehearsal
- Android Developers references on cleartext communications, Network Security Configuration, and unsafe HostnameVerifier patterns

## Prompt/Response Summary

### Step 1: Interpret the assignment
- Prompt summary: Asked AI to read the Assignment 2 spec/rubric and identify Part A case1 deliverables.
- Response summary: AI identified that Part A requires a 2-page USENIX-style report covering system/threat model, network vulnerability evidence, impact reasoning, and mitigation.
- Validation: Checked against `assignment2-spec-1.pdf` and `assignment2-rubric-1.pdf`.

### Step 2: Static analysis workflow
- Prompt summary: Asked AI to inspect the unpacked APK and decompiled source.
- Response summary: AI located the JADX cache, package `com.example.mastg_test0019`, and `MainActivity`.
- Validation: Confirmed local file `00000d58.java` contains WebView code and network-relevant logic.

### Step 3: Vulnerability discovery
- Prompt summary: Asked AI to identify insecure network behaviours.
- Response summary: AI identified `webView.loadUrl("http://www.example.com")`, `sslErrorHandler.proceed()`, and a permissive `HostnameVerifier`.
- Validation: Verified the exact code lines in decompiled `MainActivity`. Treated the HTTP load and WebView TLS error handling as the primary evidence-backed findings. Treated the HostnameVerifier as dangerous but not primary because it is not attached to a live HTTPS client in the observed path.

### Step 4: Threat model and impact
- Prompt summary: Asked AI to build a network-focused model and explain realistic impact.
- Response summary: AI framed an on-path attacker on shared Wi-Fi/rogue hotspot/captive portal and identified content injection, phishing, redirection, credential/session theft risk, and TLS bypass relevance.
- Validation: Checked that the attack capabilities match the code evidence and do not require unrelated device compromise.

### Step 5: Mitigation
- Prompt summary: Asked AI to propose concrete fixes aligned with Android guidance.
- Response summary: AI recommended enforcing HTTPS, disabling cleartext traffic, using strict Network Security Configuration, cancelling TLS errors, and removing permissive hostname validation.
- Validation: Cross-checked recommendations against Android Developers documentation.

## Rubric-Driven Mock Q&A

Q1: What is the primary vulnerability?
A1: The app loads remote WebView content over cleartext HTTP and proceeds after WebView TLS certificate errors. This breaks confidentiality, integrity, and server authentication for app-rendered remote content.

Q2: What is the attacker model?
A2: An on-path attacker between the Android app and the remote server, such as someone on the same Wi-Fi network, a rogue hotspot, captive portal, or malicious proxy.

Q3: What can the attacker do?
A3: The attacker can read or modify cleartext HTTP responses, inject fake content, redirect the user, and exploit ignored TLS errors if HTTPS content with invalid certificates is loaded later.

Q4: Why is the HostnameVerifier not the main finding?
A4: It always returns true, which is insecure, but the observed code creates it without attaching it to a live HTTPS client. The evidence-backed exploitable path is the WebView HTTP load and `sslErrorHandler.proceed()`.

Q5: What is the root-cause mitigation?
A5: Enforce HTTPS and reject TLS failures. Disabling cleartext traffic and using a strict network security configuration are defense-in-depth controls.

## Human Review / Corrections
- Avoided overclaiming the standalone `HostnameVerifier` as actively exploitable.
- Kept Part A focused only on the provided APK, as required by the spec.
- Kept the report to Tasks 1-4 and excluded Part B content.
