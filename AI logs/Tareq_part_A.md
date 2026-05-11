# INFO5995 Assignment 2 Part A - AI Usage Log

## Scope
This AI usage record only applies to the work completed for the provided APK file, `a2_case1.apk`. It does not include or describe any AI use for Part B or the in-the-wild bug bounty component.

## Tools and Materials Used
The analysis was supported by several local tools and references, including:
- `jadx` and the locally decompiled Java source code for static code analysis
- `apktool` and the unpacked APK files for checking the manifest and resources
- Local command-line searches to locate relevant evidence in the APK files
- AI assistance to understand the assignment requirements, organise the report structure, check mitigation ideas, and practise possible rubric-based questions
- Android Developers documentation related to cleartext traffic, Network Security Configuration, and unsafe hostname verification practices

## AI Prompt and Response Summary

### Step 1: Understanding the assignment requirements
- Prompt summary: I asked AI to review the Assignment 2 specification and rubric to clarify what was required for Part A, Case 1.
- Response summary: The AI explained that Part A required a short USENIX-style report covering the system and threat model, evidence of network-related vulnerabilities, impact analysis, and suitable mitigations.
- Validation: I compared this guidance with the assignment specification and rubric to confirm that the required deliverables were correctly understood.

### Step 2: Setting up the static analysis process
- Prompt summary: I asked AI to help inspect the unpacked APK and the decompiled source code.
- Response summary: The AI helped identify the relevant JADX cache, the package name `com.example.mastg_test0019`, and the main activity file used by the application.
- Validation: I manually confirmed that the decompiled Java file `00000d58.java` contained WebView-related code and network-relevant behaviour.

### Step 3: Identifying the vulnerabilities
- Prompt summary: I asked AI to identify insecure network behaviours in the decompiled application code.
- Response summary: The AI pointed out three important issues: the WebView loads `http://www.example.com`, the application calls `sslErrorHandler.proceed()`, and there is a permissive `HostnameVerifier`.
- Validation: I verified these issues directly in the decompiled `MainActivity` code. I treated the cleartext HTTP WebView load and the TLS error bypass as the main evidence-supported findings. The permissive `HostnameVerifier` was noted as insecure, but not treated as the primary exploit path because it was not clearly connected to an active HTTPS client in the observed code path.

### Step 4: Building the threat model and impact analysis
- Prompt summary: I asked AI to help explain the realistic threat model and security impact of the identified issues.
- Response summary: The AI described an on-path attacker scenario, such as an attacker on the same Wi-Fi network, a rogue hotspot, captive portal, or malicious proxy. It also explained possible impacts such as content injection, phishing, redirection, credential or session exposure, and the security risk caused by ignoring TLS errors.
- Validation: I reviewed the suggested attack model and ensured that it matched the available code evidence without assuming unrelated device compromise or unsupported attack conditions.

### Step 5: Selecting suitable mitigations
- Prompt summary: I asked AI to suggest practical mitigations that align with Android security guidance.
- Response summary: The AI recommended enforcing HTTPS, disabling cleartext traffic, applying a strict Network Security Configuration, cancelling TLS errors instead of proceeding, and removing permissive hostname verification.
- Validation: I checked these recommendations against Android Developers documentation to ensure they were appropriate and technically accurate.

## Rubric-Based Mock Q&A

**Q1: What is the main vulnerability?**  
The main vulnerability is that the app loads remote WebView content over cleartext HTTP and also proceeds when WebView TLS certificate errors occur. This weakens confidentiality, integrity, and server authentication for remote content shown inside the app.

**Q2: What attacker model applies here?**  
The relevant attacker is an on-path network attacker, such as someone on the same Wi-Fi network, a rogue hotspot operator, a captive portal, or a malicious proxy positioned between the app and the server.

**Q3: What could the attacker do?**  
The attacker could observe or modify HTTP traffic, inject fake content into the WebView, redirect the user, or take advantage of ignored TLS errors if HTTPS content with certificate problems is loaded later.

**Q4: Why is the HostnameVerifier not treated as the main finding?**  
Although the `HostnameVerifier` always returns true, which is insecure, the observed code does not clearly attach it to a live HTTPS client. Therefore, the stronger evidence-backed issue is the WebView cleartext HTTP loading and the use of `sslErrorHandler.proceed()`.

**Q5: What is the main mitigation?**  
The main mitigation is to enforce HTTPS and reject TLS failures. Disabling cleartext traffic and using a strict Network Security Configuration provide additional protection.

## Human Review and Corrections
- I avoided overstating the `HostnameVerifier` as directly exploitable because the code evidence did not show it being used in an active network path.
- I kept the analysis focused only on the provided APK for Part A, as required by the assignment.
- I limited the report scope to Tasks 1–4 and did not include Part B content.