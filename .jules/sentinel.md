# Sentinel's Journal - Critical Security Learnings Only

This journal is used to record critical security learnings discovered during the project.

| Date | Title | Vulnerability | Learning | Prevention |
| :--- | :--- | :--- | :--- | :--- |
| 2026-03-01 | Secure Local Storage & Data Isolation | Potential leakage of chat history via committed database files. | Developers often overlook local database files during environment setup, which can lead to sensitive user data being accidentally committed to the repository. | Explicitly ignore common database patterns in `.gitignore` and mandate local data encryption in `SECURITY.md`. |
