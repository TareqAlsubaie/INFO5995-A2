# INFO5995 Assignment 2 Part A - AI Usage Log

## Scope
This document records the AI-supported work completed only for the supplied APK, `a2_case1.apk`. It does not include any Part B in-the-wild bug bounty activities.

## Tools and Materials
- `jadx` cache and locally decompiled Java files used for static code analysis
- `apktool` and the extracted APK folder used to inspect the manifest and resources
- Local command-line searches used to locate relevant evidence and code references
- AI support for understanding requirements, organising the report, reviewing mitigations, and preparing practice Q&A responses
- Android Developers documentation covering cleartext traffic, Network Security Configuration, and unsafe `HostnameVerifier` usage patterns

## Prompt/Response Summary

### Step 1: Understanding the assignment
- Prompt summary: Asked AI to review the Assignment 2 specification and rubric to determine the required Part A case1 deliverables.
- Response summary: AI explained that Part A requires a two-page USENIX-style report including the system/threat model, evidence of network vulnerabilities, impact analysis, and mitigation recommendations.
- Validation: Compared the response with `assignment2-spec-1.pdf` and `assignment2-rubric-1.pdf`.

### Step 2: Static analysis process
- Prompt summary: Asked AI to examine the unpacked APK and decompiled source files.
- Response summary: AI found the JADX cache, the package `com.example.mastg_test0019`, and the `MainActivity` class.
- Validation: Verified that the local file `00000d58.java` includes WebView implementation details and network-related logic.

### Step 3: Identifying vulnerabilities
- Prompt summary: Asked AI to detect insecure network behaviours in the application.
- Response summary: AI highlighted `webView.loadUrl("http://www.example.com")`, `sslErrorHandler.proceed()`, and an overly permissive `HostnameVerifier`.
- Validation: Checked the exact lines in the decompiled `MainActivity`. The HTTP WebView load and TLS-error handling were treated as the main evidence-supported findings. The `HostnameVerifier` was considered insecure, but not the primary issue, because it was not connected to an active HTTPS client in the observed execution path.

### Step 4: Threat model and impact analysis
- Prompt summary: Asked AI to create a network-centred threat model and describe realistic security impacts.
- Response summary: AI described an on-path attacker, such as a user on the same Wi-Fi network, a rogue hotspot, captive portal, or malicious proxy, and explained risks such as content injection, phishing, redirection, credential or session exposure, and the relevance of TLS bypass.
- Validation: Confirmed that the attacker capabilities were supported by the code evidence and did not depend on unrelated device compromise.

### Step 5: Mitigation guidance
- Prompt summary: Asked AI to suggest practical fixes that align with Android security guidance.
- Response summary: AI recommended requiring HTTPS, blocking cleartext traffic, applying a strict Network Security Configuration, cancelling TLS errors instead of proceeding, and removing permissive hostname verification.
- Validation: Checked these mitigations against Android Developers documentation.

## Rubric-Driven Mock Q&A

Q1: What is the main vulnerability?  
A1: The application loads remote WebView content through cleartext HTTP and continues after WebView TLS certificate errors. This weakens confidentiality, integrity, and server authentication for remote content displayed inside the app.

Q2: What attacker model is assumed?  
A2: The assumed attacker is positioned on the network path between the Android app and the remote server. Examples include someone on the same Wi-Fi network, a rogue hotspot, a captive portal, or a malicious proxy.

Q3: What actions could the attacker perform?  
A3: The attacker could view or alter cleartext HTTP responses, inject misleading content, redirect the user, and take advantage of ignored TLS errors if HTTPS content with invalid certificates is loaded in the future.

Q4: Why is the `HostnameVerifier` not treated as the primary finding?  
A4: Although it always returns true and is therefore insecure, the observed code only creates it and does not attach it to an active HTTPS client. The clearly supported exploitable path is the WebView HTTP load together with `sslErrorHandler.proceed()`.

Q5: What mitigation addresses the root cause?  
A5: The root fix is to enforce HTTPS and reject TLS failures. Disabling cleartext traffic and using a strict Network Security Configuration provide additional defence-in-depth.

## Human Review / Corrections
- Avoided overstating the standalone `HostnameVerifier` as directly exploitable.
- Limited Part A to the provided APK, in line with the assignment specification.
- Kept the report focused on Tasks 1-4 and left out Part B material.
