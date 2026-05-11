# APK Decompilation and Network Vulnerability Analysis

## 1. Overview
This document describes the complete reverse-engineering flow for the provided Android APK, starting from decompilation through to the vulnerability finding, impact reasoning, and mitigation recommendations.

## 2. Workspace and Artifacts
- Root workspace contains `README.md`, `analysis_case1/`, and `repo_clone/`.
- `analysis_case1/jadx_out/` is the decompilation output.
- Key artifacts used in analysis:
  - `analysis_case1/jadx_out/resources/AndroidManifest.xml`
  - `analysis_case1/jadx_out/sources/com/example/mastg_test0019/MainActivity.java`
  - `analysis_case1/jadx_out/resources/res/layout/activity_main.xml`

## 3. Decompilation Process
1. Open the provided APK in Jadx or use the command-line decompiler.
2. Generate output to `analysis_case1/jadx_out`.
3. Confirm the presence of:
   - the Android manifest
   - the app resource layout files
   - the decompiled Java/Kotlin source code
4. Locate the app’s package and main activity.

## 4. Manifest Analysis
From `AndroidManifest.xml`:
- Package name: `com.example.mastg_test0019`
- Permissions:
  - `android.permission.INTERNET`
- Network security setting:
  - `android:usesCleartextTraffic="true"`
- Main launch activity:
  - `com.example.mastg_test0019.MainActivity`
- Exported activity:
  - `android:exported="true"`

### Security implications
- The app is allowed to access network resources.
- Cleartext traffic is explicitly permitted, making HTTP requests possible.

## 5. UI and Entry Point
From `activity_main.xml`:
- The UI includes a `WebView` with `android:id="@+id/webview"`.
- App text labels include:
  - `MASTG-TEST-0019`
  - `Testing Data Encryption on the Network`
- This confirms the app is designed to display web content and tests network transport behavior.

## 6. Source Code Review
From `MainActivity.java`:
- The app loads a web URL into a `WebView`:
  - `webView.loadUrl("http://www.example.com");`
- It installs a custom `WebViewClient`:
  - `onReceivedSslError(...) { sslErrorHandler.proceed(); }`
- It creates a `HostnameVerifier` that always returns `true`:
  - `verify(String str, SSLSession sSLSession) { return true; }`

### Critical insecure behaviors
- Plain HTTP is used, not HTTPS.
- SSL certificate validation is bypassed.
- Hostname verification is effectively disabled.

## 7. Full Threat Flow
1. User launches the app.
2. `MainActivity` inflates `activity_main`.
3. `WebView` is located and assigned.
4. A `WebViewClient` is attached.
5. The app calls `loadUrl("http://www.example.com")`.
6. Network traffic travels over insecure HTTP because:
   - the URL starts with `http://`
   - `usesCleartextTraffic` is enabled in the manifest
7. If HTTPS were used, SSL errors would still be ignored via `proceed()`.
8. A network attacker can:
   - read application traffic
   - modify page content
   - inject malicious JavaScript into the WebView
   - perform classic man-in-the-middle (MITM) attacks

## 8. Vulnerability Summary
The identified vulnerability class is insecure transport / TLS misconfiguration.

Evidence:
- `android:usesCleartextTraffic="true"`
- `webView.loadUrl("http://www.example.com")`
- `SslErrorHandler.proceed()` in `onReceivedSslError`
- `HostnameVerifier.verify(...)` returning `true`

These issues together allow insecure network communication and bypass HTTPS protections.

## 9. Impact Reasoning
An attacker on the same network can:
- intercept all HTTP traffic
- modify web responses before the app renders them
- inject malicious scripts or content
- compromise any data in transit
- steal session cookies if the web page relies on them

Worst-case impact:
- data disclosure
- content injection
- phishing or malicious content execution inside the WebView
- application integrity compromise

## 10. Mitigation Recommendations
1. Set `android:usesCleartextTraffic="false"` in `AndroidManifest.xml`.
2. Use HTTPS instead of HTTP:
   - `webView.loadUrl("https://www.example.com");`
3. Remove or harden SSL error handling:
   - do not call `sslErrorHandler.proceed()` for invalid certificates
4. Remove insecure `HostnameVerifier` code or implement proper verification.
5. Consider a network security configuration that disallows cleartext and enforces valid TLS.

## 11. Suggested Report Structure
1. Introduction
2. Decompilation method
3. Manifest and permissions review
4. App UI and network entry point
5. Vulnerability evidence from source code
6. Threat model and attacker capabilities
7. Impact explanation
8. Mitigations

## 12. Notes
- The vulnerability is concentrated in `MainActivity` and the `WebView` network path.
- The evidence is direct and straightforward to cite.
- This analysis covers the entire flow from APK decompilation to vulnerability identification and mitigation.
