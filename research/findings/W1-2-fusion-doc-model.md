# W1-2 — Fusion document model + workspaces + threading

**Date:** 2026-06-11
**Researcher:** workflow sub-agent (Stage 1 Wave 1)

## Summary

The Fusion API is a single-threaded, in-process object model rooted at `Application → Documents → Products → Design → Components/Occurrences`, with all cross-thread communication funneled through one primitive: `Application.fireCustomEvent`, which queues a string payload for handling on the main thread at idle. There is no public transaction API; atomicity comes only from the Command framework, which bundles everything in an `execute` handler into a single undo transaction. Design and Manufacture (CAM) workspaces are fully scriptable (CAM gained creation APIs); Render is explicitly unsupported; Simulation opened a first crack in April 2026 (Additive FEA preview). Entity identity is handled by opaque, non-comparable entityTokens resolved via `findEntityByToken`, with attributes as the durable naming alternative.

## First principles

Stripped of marketing, the Fusion API is: **one process, one thread, one object graph.** A document is a container of "products" (Design, CAM, Drawing...); the Design product owns a component tree where components hold geometry and occurrences are positioned references to them. Every mutation is an API call on the main thread; every batch boundary is a Command; every cross-thread signal is a string on an idle-time queue; every durable entity reference is a token or an attribute, never a pointer or a comparable ID.

## Findings

### 1. The authoritative object graph

Hierarchy per the official user manual: "The Application object, at the top level, represents all of Fusion" ([Getting Started with Fusion's API](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/BasicConcepts_UM.htm) — fetched 2026-06-11). A Document "represents an item in the Fusion data panel"; "Groups of related data are stored within a document as a *Product*"; Design is a Product subclass and "A document can only contain a single Design object" ([Documents, Products, Components, Occurrences and Proxies](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/ComponentsProxies_UM.htm) — fetched 2026-06-11). "Every Fusion document contains a single, default component that is referred to as the root component."

Common trip-ups, per the same page:

- **Component vs occurrence:** "A component contains geometry, whereas an occurrence has no geometry of it own, but merely displays the geometry contained in the component it references." Occurrences position/constrain; components cannot. "With the exception of the root component, a component cannot exist without at least one referencing occurrence."
- **Proxies:** "A proxy is an assembly representation of an object contained in a component" — required to disambiguate when multiple occurrences reference one component.
- **Active component is ignored by the API:** "When creating new geometry using the API, the active component is **NOT** used. Instead, new geometry is created within the component that the API is accessed from."
- **Document vs design:** `Application.activeProduct` returns whichever product matches the active workspace (the CAM product when Manufacture is active), so code that blindly casts `activeProduct` to `Design` breaks outside the Design workspace ([Introduction to the CAM API](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/CAMIntroduction_UM.htm) — fetched 2026-06-11). A document that has never activated the Manufacture workspace has no CAM product at all ([forum: Accessing CAM product](https://forums.autodesk.com/t5/fusion-api-and-scripts/accessing-cam-product/td-p/13202982) — via search, 2026-06-11, community).

### 2. Timeline / parametric history via API

Features follow the input-object pattern: create an input via `createInput`, configure with `ValueInput` objects, pass to the collection's `add` method ([BasicConcepts_UM](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/BasicConcepts_UM.htm) — fetched 2026-06-11). On edit, "properties that previously took ValueInput objects are now read-only and return a Parameter object" — you edit by changing the parameter or the feature's definition object.

Rollback and suppression are exposed on the timeline: `Timeline.markerPosition` "Gets and sets the current position of the marker"; `moveToBeginning`/`moveToEnd`; `deleteAllAfterMarker` "Deletes all objects in the timeline that are after the current position of the marker" ([Timeline Object](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Timeline.htm) — fetched 2026-06-11). Per-feature: `TimelineObject.isSuppressed` (read-write), `isRolledBack`, `rollTo(before)`, plus `healthState`/`errorOrWarningMessage` for diagnosing broken features ([TimelineObject](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/TimelineObject.htm) — fetched 2026-06-11). Editing some features requires rolling the marker to just before them, as the custom-features manual does ([TimelineObject.rollTo](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/TimelineObject_rollTo.htm) — via search, 2026-06-11).

**Transaction/batch primitive: none public, except Commands.** "Everything you do in the execute event handler is bundled within a single transaction and can be undone with one undo"; outside a command, "every API call that causes a change within Fusion will show up as a separate operation in the undo list"; "This capability is a big reason to use a command within a script." `executePreview` adds automatic rollback: "Fusion automatically aborts the previous transaction... and then you perform the creation or edit actions all over again" ([Creating Custom Fusion Commands](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Commands_UM.htm) — fetched 2026-06-11).

### 3. Workspaces: API-accessible vs closed

- **Design** — full read/write object model (the bulk of the reference manual).
- **Manufacture/CAM** — read **and write**: setups via `SetupInput`, operations via input objects, `generateAllToolpaths`/`generateToolpath`, `postProcessAll`/`postProcess`, `generateAllSetupSheets`, post-processor properties ([CAMIntroduction_UM](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/CAMIntroduction_UM.htm) — fetched 2026-06-11). No read-only restriction is stated on the current page.
- **Simulation** — closed except a fresh preview: "The API for Metal Powderbed Process Simulation is now provided as a preview feature... access and execute Process Simulation workflows programmatically" ([April 2026 Product Update — Fusion blog](https://www.autodesk.com/products/fusion-360/blog/april-2026-product-update-whats-new/), 2026-04-13 — fetched 2026-06-11 via Tavily; direct fetch 403). No general (static-stress etc.) Simulation API was found in the reference manual.
- **Render** — closed: "The API doesn't support the Render workspace. You can use the API to switch to the workspace and to call commands, including the 'In-Canvas Render' command, but without explicit API support you are limited in what you can do" ([forum accepted solution: In-Canvas render through API](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/in-canvas-render-through-api/td-p/9618616) — fetched 2026-06-11; responder identity not extractable, [unverified] whether Autodesk-badged). Workaround is `activeViewport.saveAsImageFile` plus `ui.workspaces.itemById("FusionRenderEnvironment").activate()` and executing command definitions by ID.

Workspace switching itself is API-accessible for all workspaces via `UserInterface.workspaces` / `Workspace.activate()`.

### 4. Threading and fireCustomEvent

Official position: only the main thread may touch the API. "You should not call any Fusion API functions within the worker thread. Even calling the messageBox method can sometimes result in Fusion crashing." The main thread is "a single queue where everyone has to wait their turn"; while your code runs, "nothing else is happening inside Fusion" ([Working in a Separate Thread](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Threading_UM.htm) — fetched 2026-06-11). Also: using `doEvents` "in a long term loop... will likely result in Fusion crashing at some point."

Mechanics of the only cross-thread primitive: `fireCustomEvent(eventId, additionalInfo)` where `additionalInfo` is an optional **string** ("Any additional information you want to pass through the event"). It "Returns true if the event was successfully added to the event queue" — queued, not handled: "When a custom event is fired the event is put on the queue and will be handled in the main thread when Fusion is idle"; "Firing a custom event does not immediately result in the event handler being called" ([Application.fireCustomEvent](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Application_fireCustomEvent.htm) — fetched 2026-06-11). **No payload size limit is documented**; the parameter is typed only as string. Community bridges serialize JSON into it [unverified upper bound].

### 5. Entity identity: entityTokens

Official semantics ([BRepFace.entityToken](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/BRepFace_entityToken.htm), [Feature.entityToken](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Feature_entityToken.htm) — fetched 2026-06-11):

- A token "can be saved and used at a later time with the Design.findEntityByToken method to get back the same face."
- **Tokens are not stable strings:** "the token string returned for a specific entity can be different over time," yet two different strings from the same entity both resolve to it. Therefore "you should never compare entity tokens as way to determine what the token represents" — resolve both and compare entities.
- `findEntityByToken` returns an **array**: after a face is split by later modeling, "All of the faces that represent the original face will be returned with the first face being the most logical match" ([Design.findEntityByToken](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Design_findEntityByToken.htm) — fetched 2026-06-11).
- Tokens are only valid for persisted entities ("isTemporary property is false").
- Explicit cross-session / file-round-trip durability is **not stated verbatim** on the fetched reference pages; "saved and used at a later time" implies it, and search snippets assert "Entity tokens will continue to work across sessions and when the model changes" (via search) — [unverified] pending an empirical test.

Alternatives: `tempId` is fast but B-Rep-only and "their lifetime is as long as the body remains unchanged in any way" (Brian Ekins, the API's original Autodesk designer, on the official forum; the accepted answer adds "tempId is outdated and entityToken should be used now" — community expert kandennti) ([forum: fixed ID for each edge and face](https://forums.autodesk.com/t5/fusion-api-and-scripts/can-the-fusion-360-api-create-a-fixed-id-for-each-edge-and-face/td-p/11561275) — fetched 2026-06-11 via Tavily). For durable naming, **attributes** are official: "saved by Fusion and can be retrieved later"; on B-Rep entities "attributes are never automatically deleted because the lifetime of that entity is unknown"; `findAttributes` supports regex search; attributes provide "the ability to name an entity and find it later" ([Attributes_UM](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Attributes_UM.htm) — fetched 2026-06-11).

### 6. The events surface

From the [Application object](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Application.htm) (fetched 2026-06-11) — **document lifecycle:** `documentOpening/Opened`, `documentCreated`, `documentSaving/Saved` (saving cancelable), `documentClosing/Closed` (closing cancelable), `documentActivating/Activated`, `documentDeactivating/Deactivated`; **viewport:** `cameraChanged` (rotate/zoom/pan); **app/session:** `startupCompleted`, `onlineStatusChanged`; **data/web:** `dataFileComplete`, `dataFileCopyComplete`, `insertingFromURL/insertedFromURL`, `openingFromURL/openedFromURL`, `mfgdmDataReady`.

From the [UserInterface object](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/UserInterface.htm) (fetched 2026-06-11) — **selection:** `activeSelectionChanged`; **command:** `commandCreated`, `commandStarting`, `commandTerminated`, `markingMenuDisplaying`; **workspace:** `workspacePreActivate/Activated`, `workspacePreDeactivate/Deactivated`. Command objects fire their own family (`inputChanged`, `executePreview`, `execute`, etc.) per [Events_UM](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Events_UM.htm) and [Commands_UM](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Commands_UM.htm) (fetched 2026-06-11). Plus **custom events** (Q4) as the worker-thread entry point. Known wart: `documentActivated` does not fire when switching between Design and Drawing documents ([forum bug report](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/api-bug-application-documentactivated-event-do-not-raise/td-p/9018833) — via search, community, status today [unverified]).

### 7. Units + parameter model

Internal database units are fixed: Design lengths **centimeters**, angles **radians**, mass kg — "The internal units always use these types without any exceptions." CAM differs: angles in **degrees**, feeds mm/min, time seconds ([Understanding Units in Fusion](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Units_UM.htm) — fetched 2026-06-11). "Real values are always interpreted as Fusion's database units"; string values default to document units and may embed units ("15 mm") or parameter names ([BasicConcepts_UM](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/BasicConcepts_UM.htm)).

`ValueInput.createByReal` (database units) vs `createByString` (expression: "3 in", "3/2", "hole_depth") ([ValueInput](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/ValueInput.htm) — via search 2026-06-11). User parameters are created with `UserParameters.add` (name, ValueInput, units, comment) ([UserParameters.add](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/UserParameters_add.htm) — via search); model parameters are created implicitly by features and edited via `Parameter.expression`, "a string [that] can contain any valid parameter expression." `UnitsManager` supplies `isValidExpression`, `evaluateExpression`, `formatInternalValue` for round-tripping user-facing strings ([UnitsManager](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/UnitsManager.htm) — via search). Pitfall pattern: passing a real value assuming document units (mm) yields silent 10x errors because the API reads cm; angles display as degrees to users but the API takes radians.

### 8. Direct modeling (capture-design-history off)

`Design.designType` is read/write, taking `DirectDesignType` or `ParametricDesignType`. "Changing an existing design from ParametricDesignType to DirectDesignType will result in the timeline and all design history being removed and further operations will not be captured in the timeline" ([Design.designType](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Design_designType.htm) — fetched 2026-06-11). Consequences for the API surface:

- Timeline-dependent objects (`Timeline`, `TimelineObject`, rollback, suppression) become moot — the Timeline object documents itself as representing entries "in a parametric design."
- **No parameters in direct mode** — "This was just a design choice that we made: No parameters in Direct Modeling" ([forum FR thread](https://forums.autodesk.com/t5/fusion-design-validate-document/fr-user-parameters-in-direct-modelling-please-read-before/td-p/11962690) — via search; quote attribution to an Autodesk staffer [unverified]); existing user parameters "are not lost, but are just inaccessible" ([forum](https://forums.autodesk.com/t5/fusion-design-validate-document/parametric-amp-direct-modelling-amp-user-parameters-question/td-p/9822372) — via search, community).
- Feature-creation calls still execute but produce no editable history; some APIs error in direct mode (e.g. a reported Decal RuntimeError — forum index, community, [unverified]).
- Community measurements show direct mode markedly speeds bulk API geometry creation ([New Screwdriver blog](https://newscrewdriver.com/2018/01/08/accelerate-fusion-360-api-object-creation-with-directdesigntype/) — via search, community/[empirical] third-party).
- The inverse bridge is the Base Feature: direct (non-parametric) edits inside a parametric design ([forum](https://forums.autodesk.com/t5/fusion-design-validate-document/no-design-history-captured-no-parameters/td-p/12152849) — via search, community).

## Decision implications for adze-fusion

- **The single-threaded constraint dictates bridge architecture.** Any AI sidecar (socket/MCP listener runs on a worker thread) must marshal every Fusion mutation through `fireCustomEvent` string payloads handled at idle. Latency and ordering of AI actions are governed by Fusion's idle queue, not the sidecar.
- **Atomicity requires Commands.** For "AI made an edit, user hits undo once" UX, AI-driven multi-step edits must execute inside a Command `execute` handler; raw scripted calls fragment the undo stack.
- **Reference state across an AI conversation should be entityTokens (resolve, never compare) with attributes for durable naming.** Cross-session token durability needs a Stage 4 empirical test before adze stores tokens in external memory.
- **Workspace ambition has hard edges:** Design + CAM are fully automatable (a differentiator — CAM automation is underexploited by existing MCP servers); Render is off-limits beyond command execution + screenshots; Simulation only just cracked open (Additive FEA preview, 2026-04).
- **Unit discipline is a correctness risk for LLM-generated values:** everything real-valued is cm/radians (degrees in CAM). Adze should normalize through `UnitsManager.evaluateExpression` on strings rather than trusting raw numbers.
- **Direct-mode documents silently remove the parameter/timeline surface** — adze must check `designType` before any parametric operation.

## Open questions

- Practical size ceiling of `fireCustomEvent` `additionalInfo` (undocumented; community passes JSON without published limits).
- entityToken durability across save/close/reopen and across file copies — implied but not verbatim-documented; flag for empirical verification.
- Whether the Render-workspace forum answer and the "no parameters in Direct Modeling" quote are Autodesk-badged (page chrome was lost in extraction).
- Roadmap for a general Simulation API beyond the Additive FEA preview.
- Current status of the `documentActivated` Design↔Drawing bug (reported 2020).

## Sources

- [Getting Started with Fusion's API (BasicConcepts_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/BasicConcepts_UM.htm) — fetched 2026-06-11
- [Documents, Products, Components, Occurrences and Proxies (ComponentsProxies_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/ComponentsProxies_UM.htm) — fetched 2026-06-11
- [Working in a Separate Thread (Threading_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Threading_UM.htm) — fetched 2026-06-11
- [Application.fireCustomEvent](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Application_fireCustomEvent.htm) — fetched 2026-06-11
- [Events in the Fusion API (Events_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Events_UM.htm) — fetched 2026-06-11
- [Creating Custom Fusion Commands (Commands_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Commands_UM.htm) — fetched 2026-06-11
- [Introduction to the CAM API (CAMIntroduction_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/CAMIntroduction_UM.htm) — fetched 2026-06-11
- [Timeline Object](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Timeline.htm) — fetched 2026-06-11
- [TimelineObject](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/TimelineObject.htm) — fetched 2026-06-11
- [TimelineObject.rollTo](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/TimelineObject_rollTo.htm) — via search 2026-06-11
- [Application object (events list)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Application.htm) — fetched 2026-06-11
- [UserInterface object (events list)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/UserInterface.htm) — fetched 2026-06-11
- [Feature.entityToken](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Feature_entityToken.htm) — fetched 2026-06-11
- [BRepFace.entityToken](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/BRepFace_entityToken.htm) — fetched 2026-06-11
- [Design.findEntityByToken](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Design_findEntityByToken.htm) — fetched 2026-06-11
- [Attributes in the Fusion API (Attributes_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Attributes_UM.htm) — fetched 2026-06-11
- [Understanding Units in Fusion (Units_UM)](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Units_UM.htm) — fetched 2026-06-11
- [ValueInput Object](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/ValueInput.htm) — via search 2026-06-11
- [UserParameters.add](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/UserParameters_add.htm) — via search 2026-06-11
- [UnitsManager Object](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/UnitsManager.htm) — via search 2026-06-11
- [Design.designType](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Design_designType.htm) — fetched 2026-06-11
- [April 2026 Product Update — Fusion Blog (Additive FEA API preview, Preferences API)](https://www.autodesk.com/products/fusion-360/blog/april-2026-product-update-whats-new/) — fetched 2026-06-11 via Tavily (direct fetch 403)
- [Forum: In-Canvas render through API (accepted solution: "The API doesn't support the Render workspace")](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/in-canvas-render-through-api/td-p/9618616) — fetched 2026-06-11 via Tavily
- [Forum: Can the Fusion 360 API create a fixed ID for each edge and face? (Brian Ekins on tempId; kandennti accepted answer)](https://forums.autodesk.com/t5/fusion-api-and-scripts/can-the-fusion-360-api-create-a-fixed-id-for-each-edge-and-face/td-p/11561275) — fetched 2026-06-11 via Tavily (direct fetch 403)
- [Forum: Accessing CAM product](https://forums.autodesk.com/t5/fusion-api-and-scripts/accessing-cam-product/td-p/13202982) — via search 2026-06-11
- [Forum: API BUG — Application.documentActivated does not raise](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/api-bug-application-documentactivated-event-do-not-raise/td-p/9018833) — via search 2026-06-11
- [Forum: FR — User parameters in direct modelling ("No parameters in Direct Modeling")](https://forums.autodesk.com/t5/fusion-design-validate-document/fr-user-parameters-in-direct-modelling-please-read-before/td-p/11962690) — via search 2026-06-11
- [Forum: Parametric & Direct Modelling & User Parameters](https://forums.autodesk.com/t5/fusion-design-validate-document/parametric-amp-direct-modelling-amp-user-parameters-question/td-p/9822372) — via search 2026-06-11
- [Forum: No design history captured = no parameters (Base Feature workaround)](https://forums.autodesk.com/t5/fusion-design-validate-document/no-design-history-captured-no-parameters/td-p/12152849) — via search 2026-06-11
- [New Screwdriver: Accelerate Fusion 360 API Object Creation With DirectDesignType](https://newscrewdriver.com/2018/01/08/accelerate-fusion-360-api-object-creation-with-directdesigntype/) — via search 2026-06-11 (community)
