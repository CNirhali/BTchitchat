# Palette's Journal - Critical UX/Accessibility Learnings

This journal documents critical UX and accessibility learnings discovered during the development of Bluetooth Chit Chat.

## 2025-05-14 - Semantic Markdown for Screen Readers
**Learning:** Using blockquote markers (`>`) for lists in README files is non-semantic and confusing for screen reader users, who expect `<ul>` or `<ol>` structures for list-like content. Proper Markdown lists (`-` or `1.`) provide much better navigational cues and hierarchy.
**Action:** Always use semantic list markers for features and steps, even in documentation, to ensure the "interface" is accessible to all developers.

## 2025-02-27 - DX over UX in empty repositories
**Learning:** When a repository is in an early "template" state with no application code, the "User Experience" translates directly to "Developer Experience" (DX). Improving the readability and clarity of the README is the most impactful UX contribution one can make to help the next developer get started smoothly.
**Action:** Focus on fixing formatting "noise" and run-on text in documentation when source code is absent.
