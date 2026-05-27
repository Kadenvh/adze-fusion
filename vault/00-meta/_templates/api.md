---
title: <API surface name>
category: 03-apis
tags: [api]
status: <draft | promoted>
platform: <Fusion 360 | SOLIDWORKS | Onshape | …>
namespace: <e.g. adsk.fusion>
---

# <API surface name>

<One-sentence summary.>

## First principles

<What this API REALLY exposes. Object graph, lifecycle, threading model. 4-8 sentences.>

## Surface

### Object model

<Key types and their relationships. Diagrams if useful.>

### Threading model

<STA? Event-driven? Async? What thread are callbacks invoked on?>

### Lifecycle hooks

<Connect / Disconnect / equivalent. Document-change events. Selection events.>

### Read surface

<What we can inspect — feature tree, dimensions, etc. Map to adze grounding tools.>

### Write surface

<What we can modify. Undo integration. Preview/verify patterns.>

### Limits and gotchas

<Rate limits, sandbox restrictions, version sensitivity, known bugs.>

## Implications for adze

<How this maps to adze's tool taxonomy. What ports cleanly from `C:\adze-cad` patterns vs needs rewrite.>

## Open questions

## Sources

- [Official API reference](URL) — fetched YYYY-MM-DD
- [Relevant doc / sample / blog](URL) — fetched YYYY-MM-DD

## Related

- [[platform-entry]]
- [[concept-this-illustrates]]
- [[pattern-that-uses-this-api]]
