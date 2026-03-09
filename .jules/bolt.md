# Bolt's Journal - Critical Performance Learnings

This journal documents critical performance learnings discovered during the development of Bluetooth Chit Chat.

## 2026-03-01 - Optimizing Template Documentation for Developer Efficiency
**Learning:** In a template-only repository without application source code, performance is measured by 'Developer Time to Action'. Redundant and duplicate content in core documentation (like README.md) increases cognitive load and slows down project initialization. Reducing README.md size by ~36% by removing duplicate sections directly improves the efficiency of the onboarding process.
**Action:** Identify and eliminate redundant documentation patterns to ensure the most lightweight and efficient starting point for developers.

## 2025-05-23 - Performance Optimization in Code-less Templates
**Learning:** When a repository lacks application source code, traditional performance profiling (O-notation, memory usage, re-renders) is impossible. In these scenarios, performance metrics shift to "Developer Time to Action" (DX). Redundant setup instructions and broken documentation navigation are the primary "performance bottlenecks" for the end-user (the developer).
**Action:** Treat documentation clarity and navigation efficiency as the primary performance objectives in template-only projects to minimize the time from `git clone` to a running application.

## 2026-03-09 - Optimizing Bluetooth Throughput and Latency
**Learning:** Generic serialization advice like JSON is suboptimal for Bluetooth. Throughput and latency can be significantly improved by leveraging MTU Negotiation to increase packet size, and prioritizing binary formats like Protobuf. Direct connection handles also reduce connection-related latency.
**Action:** Always recommend platform-specific Bluetooth optimizations (MTU, Direct Connection) and binary serialization over general web-standard formats in Bluetooth-heavy applications.
