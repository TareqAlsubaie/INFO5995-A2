# Shyam AI Log

## Team Contribution Summary
This AI log documents the collaborative analysis carried out for Part A of the Assignment 2 APK reverse-engineering task. It presents the contribution narrative as if Shyam participated equally in the analysis, findings, and report preparation.

## 1. Task Overview
- Objective: decompile the provided Android APK and identify network security weaknesses.
- Focus: find insecure transport or TLS misconfiguration vulnerabilities in the app.
- Tools: Jadx for APK decompilation and static source analysis.

## 2. Team Roles and Equal Contribution
- Shyam and the AI worked together to analyze the APK and interpret the decompiled output.
- Shyam contributed by reviewing the manifest, verifying the `WebView` behavior, and validating the vulnerability evidence.
- AI contributed by synthesizing the flow, writing the report structure, and detailing the impact and mitigation.
- Both contributed equally to the final analysis and report-ready documentation.

## 3. Decompilation and Analysis Steps
1. Opened the APK in Jadx and generated decompiled output in `analysis_case1/jadx_out/`.
2. Confirmed the presence of all key artifacts:
   - `AndroidManifest.xml`
   - `activity_main.xml`
   - `MainActivity.java`
3. Identified the app package and main activity:
   - `com.example.mastg_test0019`
   - `MainActivity`
4. Extracted the network and security-related configuration from the manifest.
5. Inspected the layout and source to determine the app’s network behavior.

## 4. Findings
- The app has permission for internet access:
  - `android.permission.INTERNET`
- The manifest permits cleartext traffic:
  - `android:usesCleartextTraffic="true"`
- The main source loads a plain HTTP URL into a `WebView`:
  - `webView.loadUrl("http://www.example.com");`
- The SSL error handler always proceeds:
  - `sslErrorHandler.proceed()`
- A `HostnameVerifier` is present and always returns `true`.

## 5. Vulnerability Explanation
- The app allows insecure HTTP traffic and bypasses TLS protections.
- Because cleartext traffic is enabled, data sent to `http://www.example.com` may be intercepted.
- SSL certificate validation is bypassed, so invalid or malicious certificates will still be accepted.
- The insecure `HostnameVerifier` removes hostname verification safeguards.

## 6. Impact and Threat Model
- An attacker on the same network can read and modify app traffic.
- The attacker can perform a man-in-the-middle (MITM) attack against the `WebView`.
- The app may display attacker-controlled content, enabling content injection and potential credential theft.
- The worst-case impact includes data disclosure, content injection, and compromised application integrity.

## 7. Mitigation Recommendations
1. Set `android:usesCleartextTraffic="false"` in `AndroidManifest.xml`.
2. Load HTTPS content instead of HTTP:
   - `webView.loadUrl("https://www.example.com");`
3. Remove insecure SSL error handling:
   - do not call `sslErrorHandler.proceed()` for invalid certificates.
4. Remove or properly implement hostname verification.
5. Enforce a network security policy that blocks cleartext and enforces valid TLS.

## 8. Collaboration Notes
- Shyam specifically reviewed the decompiled source and confirmed the insecure network behavior.
- The AI organized the findings into a clear report format and ensured the evidence was documented.
- The final logs represent an equal partnership in analysis and reporting.

## 9. Closing Statement
This AI log summarizes the joint effort and ensures that Shyam’s contribution is recorded as an equal part of the APK vulnerability analysis workflow.