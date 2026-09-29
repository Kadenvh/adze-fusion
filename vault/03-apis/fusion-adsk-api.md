---
title: Fusion adsk.* Python API
category: 03-apis
tags: [api, fusion-360, python, addin]
status: promoted
confidence: medium
platform: Fusion 360
namespace: adsk
---

# Fusion adsk.* Python API

Fusion 360's native in-process API exposed as the `adsk.*` Python namespace. Single-threaded with UI ownership; mutations marshal to the main thread via `CustomEvent`.

## First principles

The `adsk` Python module is loaded into a Python 3.7 interpreter embedded inside the Fusion process — there is no IPC for the add-in itself; the API is direct call into Fusion's address space. The two consequences that drive every architecture choice: (1) only one thread (Fusion's main thread) may touch the document model, so any worker thread must marshal via `app.fireCustomEvent`; (2) the entire add-in dies when Fusion exits, so persistent state (cache, learned context, history) lives outside the add-in.

The API is structured as `adsk.core` (host/app/UI primitives), `adsk.fusion` (the parametric modeling object graph), and `adsk.cam` (CAM workspace). Add-in lifecycle hooks are `run(context)` / `stop(context)` — invoked by Fusion when the add-in is loaded or unloaded.

## Surface

### Object model

`Application` (singleton) → `Documents` → `Document` → `Product` (Design / CAM / etc.) → workspace-specific roots (e.g., `Design.rootComponent`, then `Component → Occurrences / Features / Sketches / BRepBodies`). Parametric history is first-class — every feature is an object in the timeline.

### Threading model

Single-threaded. UI events and document mutations both run on Fusion's main thread. The canonical pattern for "external process drives Fusion": (a) worker thread (TCP listener, MCP stdio bridge) lives inside the add-in's Python; (b) on incoming request, worker calls `app.fireCustomEvent(eventId, jsonPayload)`; (c) a `CustomEventHandler` registered on the main thread runs the actual modeling code; (d) response is shipped back via a future/queue. [[../05-mcp-servers/faust-machines-fusion360-mcp]] is the canonical reference implementation.

### Lifecycle hooks

- `run(context)` — called when add-in is started. Set up handlers, register commands.
- `stop(context)` — called on unload. Tear down handlers; Fusion will kill the process if this misses.
- `CommandCreated` / `CommandExecute` events — UI integration.
- Document-change events — `DocumentSaving`, `DocumentClosed`, etc.

### Read surface

Feature tree (timeline), dimensions, parameters (`design.userParameters`), BRep geometry (`BRepBodies`, `BRepFaces`, `BRepEdges`), sketches (`Sketches` → `SketchEntities`), assembly graph (`Occurrences`), mass properties (`PhysicalProperties`).

### Write surface

Sketches and features via the `Design.rootComponent.<features>.add()` collections; parameter mutation via `userParameter.expression = "..."`; export via `ExportManager`. Undo is automatic per command — wrap a logical operation in a `CommandDefinition` to get one undo step.

### Limits and gotchas

- Python 3.7 only — no f-string walrus, no `match`, no modern type-hint syntax.
- `BRepBody` references invalidate across timeline edits; long-lived references must be re-fetched by token.
- HTTP servers run in worker threads only; touching `app.activeDocument` from a worker without `CustomEvent` is a crash class. `[unverified]` — pattern documented by community servers but no Autodesk-official statement was fetched in P1.

## Implications for adze

- **The shared-core port boundary is the worker/main-thread split.** Anything that mutates state goes through `CustomEvent`; anything that reads inert data can read from the worker. This maps cleanly to adze-cad's write-safety lifecycle.
- **Long-lived geometry references are unsafe.** Cache by token (`entityToken`), not by Python object. This differs from SOLIDWORKS COM identity stability and is a real port concern.
- **Undo integration via `CommandDefinition`** is the canonical "one logical operation = one undo step" pattern. Adze-cad's recipe-capture model maps onto this.

## Open questions

- Exact list of which events fire on which thread — community sources don't enumerate, and the official API reference returned 403 in P1.
- Whether `adsk.fusion` exposes a stable diff/patch surface for parametric history (relevant to recipe replay across sessions).
- Whether the embedded interpreter ever upgrades from Python 3.7.

## Sources

- [Fusion Help — Python Specific Issues](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — fetched 2026-05-15 (via search)
- [faust-machines/fusion360-mcp-server — README](https://github.com/faust-machines/fusion360-mcp-server) — fetched 2026-05-15 (CustomEvent main-thread pattern documented)
- [Driving Fusion via AI using MCP server add-in — Autodesk Community forum (403)](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/driving-fusion-via-ai-using-mcp-server-add-in-announcement/td-p/13881165) — referenced in P1; full content not fetched

## Related

- [[../02-platforms/fusion-360]]
- [[../07-patterns/in-process-addin-localhost-mcp]]
- [[../05-mcp-servers/faust-machines-fusion360-mcp]]
- [[../05-mcp-servers/joe-spencer-fusion-mcp]]
