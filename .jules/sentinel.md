# Sentinel's Journal - Critical Security Learnings Only

This journal is used to record critical security learnings discovered during the project.

| Date | Title | Vulnerability | Learning | Prevention |
| :--- | :--- | :--- | :--- | :--- |
| 2026-03-01 | Secure Local Storage & Data Isolation | Potential leakage of chat history via committed database files. | Developers often overlook local database files during environment setup, which can lead to sensitive user data being accidentally committed to the repository. | Explicitly ignore common database patterns in `.gitignore` and mandate local data encryption in `SECURITY.md`. |
| 2026-03-04 | Standardizing Bluetooth Security Guidelines | Lack of explicit guidance on pairing security, resource limiting, and data minimization in Bluetooth communication. | In template-based projects, security guidelines often focus on general security while overlooking technology-specific risks like Bluetooth MITM or DoS. | Always include technology-specific security risks in the project's security policy to guide implementers. |
| 2026-03-05 | Implementing Discovery Timeouts | Perpetual Bluetooth discoverability increases the device's attack surface to any nearby malicious actor. | Many Bluetooth implementations forget to disable discoverability once a connection is established or after a certain period, leaving the device unnecessarily exposed. | Mandate discovery timeouts in the security guidelines to ensure the window of exposure is minimized. |
