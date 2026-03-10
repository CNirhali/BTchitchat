# 🛡️ Security Policy

## 📋 Table of Contents

- [🗓️ Supported Versions](#supported-versions)
- [🛡️ Security Development Guidelines](#security-development-guidelines)
- [🚨 Reporting a Vulnerability](#reporting-a-vulnerability)

## 🗓️ Supported Versions

Currently, we are in active development. Only the latest version of the application is supported.

| Version | Supported          |
| ------- | ------------------ |
| < 1.0.0 | :white_check_mark: |

## 🛡️ Security Development Guidelines

To maintain the security of the Bluetooth Chit Chat application, all contributors must follow these guidelines:

- ⚖️ **Input Validation:** Sanitize and validate all data received over Bluetooth before processing or displaying it. Bluetooth packets can be manipulated by malicious devices.
- 🔑 **No Hardcoded Secrets:** Never include API keys, passwords, or other sensitive credentials in the source code. Use environment variables or secure storage mechanisms.
- 🛡️ **Principle of Least Privilege:** Only request the minimum necessary Bluetooth and Location permissions required for the app to function.
- 💾 **Secure Data Storage:** Encrypt chat logs and other sensitive local data. Use platform-specific secure storage (e.g., Keystore for Android, Keychain for iOS) for managing encryption keys.
- ⚠️ **Secure Error Handling:** Ensure that error messages do not leak sensitive information or stack traces. Log detailed errors internally, but provide generic messages to the user.
- 🤝 **Secure Pairing & Authentication:** Implement secure pairing mechanisms (e.g., Numeric Comparison) and authenticate connected devices to prevent unauthorized access and MITM attacks.
- 🚦 **Resource Limits & Rate Limiting:** Apply limits on message sizes and frequency of incoming Bluetooth packets to prevent denial-of-service (DoS) and memory exhaustion.
- 🤏 **Data Minimization:** Only transmit essential data over Bluetooth. Avoid sending Personally Identifiable Information (PII) unless it is strictly necessary and properly encrypted.
- ⌛ **Replay Protection:** Implement nonces or timestamps to prevent captured Bluetooth packets from being re-sent to the application.
- 🎲 **Cryptographically Secure Randomness:** Use platform-provided cryptographically secure random number generators (CSPRNGs) for generating all cryptographic keys, nonces, and session identifiers.
- 📍 **Bluetooth Discoverability:** Implement a timeout for discoverability to minimize the window of exposure to unknown devices.
- 🚫 **Data Leakage Prevention (DLP):** Prevent sensitive data leakage through unencrypted cloud backups (e.g., `android:allowBackup="false"`), the system clipboard, by obscuring sensitive UI content in the application switcher, or by disabling screenshots on sensitive screens (e.g., `FLAG_SECURE` on Android).
- 📝 **Secure Logging:** Do not log Personally Identifiable Information (PII) or sensitive message content. Use a production-ready logging library that supports log level filtering.
- 🔔 **Secure Notifications:** Avoid displaying sensitive chat content in system notifications that appear on the lock screen. Use generic summaries (e.g., "New Message") instead.
- 🔗 **Dependency Security:** Regularly audit and update third-party libraries to mitigate risks from known vulnerabilities. Use tools like `pnpm audit` or `snyk` to automate this process.
- 🌐 **Secure Network Communication:** Ensure all network traffic uses encrypted protocols (e.g., HTTPS). Disable cleartext traffic in the application configuration (e.g., `android:usesCleartextTraffic="false"`).
- ⚔️ **Anti-Tampering:** Implement root and jailbreak detection mechanisms to prevent the application from running on compromised devices, protecting its sandbox and user data.

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>

## 🚨 Reporting a Vulnerability

Security is a top priority for this project. If you discover a security vulnerability, please do not open a public issue. Instead, please report it through the following process:

1. 📧 **Email**: Send a detailed report to [chaitanyanirhali@gmail.com](mailto:chaitanyanirhali@gmail.com).
2. 📝 **Details**: Include a description of the vulnerability, steps to reproduce, and any potential impact.
3. ⌛ **Response**: You can expect an acknowledgment of your report within 48 hours. We will keep you updated on the progress of the fix.

We appreciate your help in keeping this project secure!

---

<a href="#security-policy" aria-label="Back to top of page">⬆ Back to Top</a>
