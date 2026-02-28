# Palette's Journal - Critical UX/Accessibility Learnings

This journal documents critical UX and accessibility learnings discovered during the development of Bluetooth Chit Chat.

## 2025-05-14 - Semantic Markdown for Screen Readers
**Learning:** Using blockquote markers (`>`) for lists in README files is non-semantic and confusing for screen reader users, who expect `<ul>` or `<ol>` structures for list-like content. Proper Markdown lists (`-` or `1.`) provide much better navigational cues and hierarchy.
**Action:** Always use semantic list markers for features and steps, even in documentation, to ensure the "interface" is accessible to all developers.
