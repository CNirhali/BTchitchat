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

## 2026-03-10 - Browser Rendering Optimization for Documentation
**Learning:** Even in a template-only repository, web performance of the documentation (the primary product) can be improved. Using `loading="lazy"` on heavy placeholder images reduces the initial page load payload and improves browser rendering speed for developers reviewing the project.
**Action:** Always include `loading="lazy"` for images in documentation to ensure the best possible web performance for the project's landing page.

## 2026-03-11 - Holistic Documentation Performance: Rendering, Navigation, and Technical Guidelines
**Learning:** Documentation performance in template repositories is multi-faceted. It involves optimizing browser rendering using `decoding="async"` to prevent main-thread blocking, improving navigation ergonomics with contextual "Back to Top" links to reduce scroll-time, and providing high-impact technical patterns (like Object Pooling and Lazy Initialization) that guide developers toward building high-performance applications from the start.
**Action:** Combine browser-level rendering optimizations with structural UX improvements and actionable performance patterns to maximize developer productivity and application quality.

## 2026-03-15 - LCP Optimization and Specialized Bluetooth Performance Patterns
**Learning:** General performance advice like `loading="lazy"` should be avoided for above-the-fold content in documentation to optimize Largest Contentful Paint (LCP). Furthermore, specialized Bluetooth patterns such as GATT Caching and Filtered Scanning are high-impact optimizations that significantly reduce "time-to-first-chat" and system overhead, which are critical for the UX of Bluetooth-based messaging applications.
**Action:** Always audit documentation for LCP-critical images and prioritize platform-specific Bluetooth optimizations (like GATT Caching) in performance guidelines to ensure developers start with the most efficient patterns.

## 2026-03-16 - Documentation-Driven Performance in Template-Only Repositories
**Learning:** In repositories without application code, performance optimization is achieved through a multi-layered approach: improving browser rendering of documentation (e.g., using `fetchpriority="high"` for LCP), enhancing developer navigation ("Back to Top" links), and providing actionable technical snippets (e.g., MTU Negotiation, Background Threading) that ensure the final product built from the template is performant by design.
**Action:** When working in template-only environments, prioritize technical implementation examples within documentation to satisfy the requirement for functional performance enhancements.

## 2026-03-22 - Platform Parity in Performance Documentation
**Learning:** In template repositories, performance is synonymous with 'Developer Time to Action'. When high-impact optimizations like MTU Negotiation and GATT Caching are only provided for one platform (e.g., Android), it creates a performance gap for developers on other platforms (e.g., iOS). Providing Swift equivalents with optimized defaults (like `.withoutResponse`) ensures all developers can achieve peak performance regardless of their chosen stack.
**Action:** Always ensure technical implementation examples have platform parity (Kotlin/Swift) and use the most performant API configurations (e.g., write-without-response) as the documented default.

## 2026-03-17 - Protobuf Runtime Optimization for Mobile
**Learning:** For mobile applications, the default Protobuf runtime can be unnecessarily heavy due to reflection and descriptors. Using `option optimize_for = LITE_RUNTIME;` in the schema significantly reduces the binary footprint and memory usage of the generated code, which is critical for performance-constrained environments like Bluetooth-based messaging.
**Action:** Always consider `LITE_RUNTIME` for Protobuf schemas intended for mobile or embedded platforms where reflection is not strictly required.

## 2026-03-18 - Protocol-Level Efficiency: Heartbeats and Binary Serialization
**Learning:** Connection maintenance in Bluetooth-based apps often requires periodic "pings." Reusing a full `ChatMessage` for this purpose is inefficient. Adding a specialized `HEARTBEAT` message type to the Protobuf enum allows for a minimal 2-byte payload, significantly reducing radio activity and battery drain compared to generic message types.
**Action:** Always include specialized, minimal message types for protocol-level signals (heartbeats, ACKs) in low-bandwidth communication schemas to maximize battery life and minimize packet overhead.

## 2026-03-19 - Protobuf Parsing and Memory Optimization
**Learning:** Protobuf parsing efficiency can be improved by reordering fields in the schema to match their tag numbers (sequential order), as many runtimes process tags more efficiently this way. Furthermore, for C++ consumers, enabling arena allocation (`cc_enable_arenas = true`) significantly reduces memory fragmentation and allocation overhead during high-frequency message processing.
**Action:** Always order Protobuf fields sequentially by tag number and enable arena allocation for performance-critical communication schemas to maximize runtime efficiency and minimize memory pressure.

## 2026-03-20 - Documentation-Driven Performance: Filtered Scanning and Navigation Optimization
**Learning:** In template-only repositories, performance is defined by 'Developer Time to Action'. Providing high-impact, actionable code snippets for specialized operations like Filtered Scanning and immediate discovery termination directly improves the quality and efficiency of the resulting application. Additionally, structural UX improvements like contextual 'Back to Top' links reduce developer cognitive load and navigation time.
**Action:** Always prioritize including platform-specific, copy-pasteable best practices in documentation and ensure seamless navigation to maximize both developer and application performance.

## 2026-03-21 - DX Performance: Reducing Onboarding Friction with Actionable Snippets
**Learning:** In template repositories, performance is synonymous with 'Developer Time to Action'. Providing platform-specific (Kotlin/Swift) code snippets for critical operations like Message Batching and Background Threading significantly reduces the cognitive overhead and implementation time for developers. Parity across platforms is essential to ensure a consistent performance baseline regardless of the chosen stack.
**Action:** When source code is absent, prioritize adding technical implementation examples for all high-impact performance guidelines to ensure developers can implement optimizations immediately.

## 2026-03-23 - Optimizing Algorithm Efficiency and Platform Parity in Documentation
**Learning:** Documentation snippets for performance-critical logic, such as Replay Protection, must use efficient data structures to avoid hidden O(n) bottlenecks (e.g., Swift's `removeFirst()` on an Array). Furthermore, ensuring platform parity for high-throughput defaults (like `WRITE_TYPE_NO_RESPONSE` on Android) ensures that developers on all platforms achieve the same performance baseline by default.
**Action:** Audit documentation snippets for O(n) operations and ensure that performance-critical API defaults are consistently applied across all supported platforms.

## 2026-03-24 - Protobuf Builder Pooling and Capacity Reuse for Mobile Performance
**Learning:** In high-frequency Bluetooth messaging, object allocation is a primary source of UI jank. For Kotlin/Protobuf, pooling `Builder` objects instead of immutable `ChatMessage` objects is the correct pattern. On iOS/Swift, leveraging `removeAll(keepingCapacity: true)` for message buffers prevents expensive re-allocations of the underlying array storage. These micro-optimizations significantly reduce GC pressure and improve frame rates.
**Action:** Always recommend pooling mutable builders and reusing collection capacity in performance-critical data paths to ensure smooth UI performance and efficient memory usage.

## 2026-03-26 - DX Performance: Optimizing Service Discovery and Documentation Efficiency
**Learning:** In template-only repositories, 'Developer Time to Action' (DX) is the primary performance metric. Redundant sections and broken navigation links in the documentation act as performance bottlenecks for the developer. Furthermore, promoting partial service discovery (e.g., `peripheral.discoverServices([serviceUUID])`) instead of full discovery (e.g., `peripheral.discoverServices(nil)`) provides a high-impact optimization for both connection speed and battery efficiency.
**Action:** Audit documentation for redundancy and navigation issues to improve DX, and always prioritize targeted Bluetooth operations over generic, broad ones in technical snippets.

## 2026-03-27 - Maximizing Bluetooth Throughput and Minimizing Connection Latency
**Learning:** Default Bluetooth connection parameters often incur significant overhead. Specifying `TRANSPORT_LE` on Android avoids dual-mode (BR/EDR) negotiation, reducing connection delay. For throughput, requesting **2M PHY** (BLE 5.0+) doubles the bit rate, while implementing **Write Without Response flow control** on iOS (via `canSendWriteWithoutResponse`) allows for maximum saturation of the Bluetooth link without packet loss or buffer overflows.
**Action:** Always provide specialized Bluetooth optimizations like `TRANSPORT_LE`, 2M PHY, and flow control in performance guidelines to ensure developers can achieve peak hardware performance.

## 2026-03-28 - Verifying Documentation Snippets for DX Performance
**Learning:** When providing technical snippets in documentation, omitting variable declarations or context (like `characteristic` or `gatt` instance) severely degrades 'Developer Time to Action' (DX). Developers expect copy-pasteable or easily adaptable code; incomplete snippets force them to search for missing context, defeating the purpose of the optimization.
**Action:** Always include necessary parameters and context in documentation snippets, and double-check for variable consistency and existence before finalizing changes.
