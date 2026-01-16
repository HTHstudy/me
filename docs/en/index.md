---
layout: default
title: English
nav_order: 2
has_children: true
permalink: /docs/en
---

# React, but with Reasons

🌐 **English** | [한국어](../ko)

---

This is a personal portfolio about **how I reason about React and frontend architecture**.

It is not a tutorial, and it is not a collection of best practices.
Instead, it documents:

- how I frame frontend problems,
- how I evaluate trade-offs,
- and how I make architectural decisions under real-world constraints.

If you are interested in _why_ certain patterns work (or fail),
this is a good place to start.

---

## How this repository is organized

### Mental Model

How I conceptualize frontend systems _before_ writing code.

This section focuses on:

- separation of concerns as a practical tool,
- UI logic vs domain logic,
- headless UI and boundary-driven design,
- accessibility as a core design constraint,
- and responsibility boundaries in frontend systems.

→ Recommended starting point if you want to understand **how I think**.

---

### Decisions over Patterns

Patterns are outcomes, not starting points.

This section explores:

- controlled vs uncontrolled components,
- state management decision criteria,
- why React Context is not global state,
- using Context as scoped dependency injection (DI),
- and choosing the right React primitive for a given problem.

→ Recommended if you care about **React-specific design decisions**.

---

### Anti-patterns I’ve seen in the wild

Common failure modes that emerge over time.

This section documents:

- callback ref misuse and hidden coupling,
- implicit state and flag-driven logic,
- boundary leaks between UI, domain, and infrastructure,
- and how StrictMode exposes fragile designs.

These are not “rules”, but observations about **when costs begin to outweigh benefits**.

---

### Case Studies

Anonymized, real-world refactoring stories.

Each case study focuses on:

- what the original design optimized for,
- where it started to break down,
- what changed,
- and what trade-offs remained after refactoring.

(This section grows over time.)

---

## How to read this

- Start with **Mental Model** for the big picture
- Jump to **Decisions over Patterns** for concrete React examples
- Read **Anti-patterns** to understand common pitfalls
- Use **Case Studies** for practical, experience-driven insights
