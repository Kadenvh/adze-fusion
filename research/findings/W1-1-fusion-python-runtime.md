# W1-1 — Fusion Python add-in API: runtime, lifecycle, packaging, install path

**Date:** 2026-06-11
**Researcher:** workflow sub-agent (Stage 1 Wave 1)

## Summary

Fusion's embedded Python is **3.14 as of January 2026** — the prior finding of "3.7" is four years stale (3.7 ended in early 2022). Add-ins are folders containing a same-named `.py` entry point exposing `run(context)`/`stop(context)` plus a JSON `.manifest`; Fusion auto-discovers them in several documented per-user locations: the `API/AddIns` directory, the `FusionAddins` directory (`%appdata%\Autodesk\FusionAddins` on Windows; `~/Library/Application Support/Autodesk/FusionAddins` on Mac), and — for store apps — the `ApplicationPlugins` bundle directories. The cited doc page is internally inconsistent on the `API/AddIns` parent folder name (`Autodesk Fusion` vs `Autodesk Fusion 360`), so neither name is authoritative pending an [empirical] on-disk check. App Store distribution uses a different layout entirely: a `.bundle` folder with `PackageContents.xml` installed into `ApplicationPlugins` by an Autodesk-built installer. The API is strictly main-thread; worker threads marshal back via `fireCustomEvent`. Vendoring third-party modules beside the script is the officially recommended packaging method, and VS Code is the official editor/debugger.

## First principles (what the thing IS, stripped of marketing)

A Fusion Python add-in is **a folder of interpreted source files executed inside Fusion's own process, on Fusion's own embedded CPython interpreter, on Fusion's single main UI thread**. There is no separate runtime, no compiled artifact requirement, no plugin registry — discovery is "files in a known directory at startup." The `.manifest` is a small JSON descriptor (metadata + load behavior), not a permission or capability declaration. The lifecycle is two functions: Fusion calls `run(context)` at load, `stop(context)` at unload. Everything else (UI, events, threads) is built on top of that inside the host process.

## Findings

### 1. Embedded Python version (2026): 3.14 — prior "3.7" finding is wrong/stale

The official 2026 Release Notes, under **Build v.2606.1.22 (January 21, 2026)**, API section: "Updated the embedded Python version from 3.12 to 3.14." ([2026 Release Notes](https://help.autodesk.com/cloudhelp/ENU/Fusion-ReleaseNotes/files/FIC-REL-NOTES-INC.htm) — fetched 2026-06-11). The February 2026 notes add: "The Haas Driver, Haas Tooling and Renishaw Fixturing and Styli add-Ins have been updated from Python 3.12 to Python 3.14" (same page).

The "3.7" figure dates to before **February 2022**, when Autodesk staff announced the move to 3.9.7: "In the next MAJOR Fusion 360 update the embedded python interpreter will be upgraded to Python 3.9.7… Add-ins compiled with the current version of Python (3.7) will not be compatible" ([Important Python Version Change Information — Autodesk forum, official Autodesk post](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/important-python-version-change-information/td-p/10929351) — fetched 2026-06-11). Trajectory: 3.7 → 3.9.7 (2022) → 3.12 → 3.14 (Jan 2026). The current Python Specific Issues help page no longer states a version at all ([Python Specific Issues](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — fetched 2026-06-11). Whether Fusion ships the standard GIL build or 3.14's free-threaded variant is not documented [unverified; assume standard GIL build].

### 2. Lifecycle, manifest format, entry-point structure

All from [Creating a Script or Add-In](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/WritingDebugging_UM.htm) — fetched 2026-06-11.

**Lifecycle.** `run(context)` is called when the add-in starts; `stop(context)` "is called by Fusion whenever the add-in is being stopped and unloaded. This can happen because the user is stopping it using the 'Scripts and Add-Ins' dialog or more typically it is because Fusion is shutting down and all add-ins are being stopped." Both take a single `context` argument — "Python passes this in as a Dictionary object and C++ passes it in as a string in JSON format." Documented keys: for `run`, `IsApplicationStartup` ("Indicates the add-in is being started as a result [of] automatic loading during Fusion startup (true) or is being loaded by the user through the 'Scripts and Add-Ins' dialog (false)"); for `stop`, `IsApplicationClosing` (true when Fusion is shutting down).

**Folder/entry-point structure.** An add-in is a folder named for the add-in containing `<Name>.py` (the entry point, same basename as the folder) and `<Name>.manifest`; "Additional files associated with the script or add-in (icons, for example) should be added to this folder."

**Manifest.** A JSON text file with these documented fields: `autodeskProduct` (product identifier; the exact current string is reported both as "Fusion360" and "Fusion" across doc renderings [unverified verbatim]), `type` ("can be 'addin' or 'script'"), `id` (GUID; current docs say it can be left empty), `author` (shown in the dialog), `description` (JSON object keyed by language code, multilingual), `version` ("can be any string, i.e. '1.0.0', '2016', 'R1', 'V2', etc."), `runOnStartup` (true/false), `supportedOS` ("windows", "mac", or "windows|mac"), `editEnabled`, `iconFilename`.

### 3. Install paths and discovery

Verbatim from the current API user manual ([Creating a Script or Add-In](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/WritingDebugging_UM.htm) — fetched 2026-06-11):

- Add-ins, Windows: `%appdata%\Autodesk\Autodesk Fusion\API\AddIns`
- Add-ins, Mac: `$HOME/Library/Application Support/Autodesk/Autodesk Fusion/API/AddIns`
- Scripts: same parents with `\API\Scripts`.

Discovery rule, quoted: "Scripts and add-ins can exist at any location on the machine but it's only in the locations listed above where Fusion automatically searches for add-ins when it starts up. A script or add-in in any other location will need to be explicitly located using the green '+' icon near the top of the 'Scripts and Add-Ins' dialog." The default path is user-configurable: "you can edit the default path in the 'General' -> 'API section' in the Preferences command."

The Apr 2026 support article confirms the same paths ([How to install an add-in or script in Autodesk Fusion](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-to-install-an-ADD-IN-and-Script-in-Fusion-360.html) — fetched 2026-06-11 via Tavily; WebFetch 403). **Caution:** older docs and most community material use `Autodesk Fusion 360` in the path; current official pages say `Autodesk Fusion`. Which folder name exists on a given install (rename vintage, symlinks) needs an [empirical] check on a real machine — not verified here.

Separately, App Store apps are discovered in the **ApplicationPlugins** bundle directories (see Q4): Windows `%APPDATA%\Autodesk\ApplicationPlugins`; Mac (web-installed Fusion) `~/Library/Application Support/Autodesk/ApplicationPlugins`; Mac (Mac App Store Fusion) `~/Library/Containers/com.autodesk.mas.fusion360/Data/Library/Application Support/Autodesk/ApplicationPlugins` — "Fusion is looking for add-in bundles to load" in these directories; the MAS path differs because "all MAS apps run in a sandbox environment, and they cannot load additional modules from outside their folder structure without user interaction" ([Fusion app paths — Autodesk Developer Blog](https://blog.autodesk.io/fusion-app-paths/) — fetched 2026-06-11).

### 4. How users install: App Store vs manual

**Manual:** download from GitHub/etc., copy the folder into the `API/AddIns` path above (support article steps), or link an arbitrary location via the green "+" in the Scripts and Add-Ins dialog. ([Support article](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-to-install-an-ADD-IN-and-Script-in-Fusion-360.html), fetched 2026-06-11.)

**App Store:** the user downloads an OS-specific installer; "If the script or add-in is in an installer from the app store, run the installer for the relevant add-in. Verify that you are installing the correct version for your OS" (same article). On the publisher side, "Your add-in must be in a `.bundle` folder containing: PackageContents.xml (configuration file) [and] a Contents subfolder with your deliverables," and "We strongly recommend you make use of the standard… Autodesk App Store app installer we create for you. Installers can be created for Windows and Mac"; "Apps installed in the per-user folder are only available to that user" ([Fusion publishing guidelines — APS Publisher Center](https://aps.autodesk.com/app-store/publisher-center/fusion-360) — fetched 2026-06-11). **What it does on disk:** places the `.bundle` (PackageContents.xml + Contents/) into the per-user `ApplicationPlugins` folder for the platform (paths in Q3) ([Autodesk Developer Blog](https://blog.autodesk.io/fusion-app-paths/)). Publisher guidelines also impose a load-time budget ("under 0.005 seconds" cited for command/load efficiency, per the same guidelines page). A claim that App Store installs "require elevated (admin) user privileges" appears in ADN publisher FAQ material (via search) but conflicts with the per-user install location — [unverified].

### 5. Threading constraints (official)

From [Working in a Seperate Thread — Fusion Help](https://help.autodesk.com/view/fusion360/ENU/?guid=GUID-F9FD4A6D-C59F-4176-9003-CE04F7558CCC) — fetched 2026-06-11 via Tavily (WebFetch 404 on the SPA page):

- "The Fusion API doesn't support the creation or management of threads" — you use the language's own threading.
- "you should not call any Fusion API functions within the worker thread. Even calling the messageBox method can sometimes result in Fusion crashing."
- "Your worker thread should never do any work inside Fusion because that should always be done in the main thread." / "The main thing is that all Fusion specific work needs to be done by your add-in in the main thread."
- Pattern: register a custom event + handler, start the worker thread, and the worker "calls the fireCustomEvent method. This results in Fusion calling the event handler for the custom event… This gives the add-in control of the main thread."

`fireCustomEvent` mechanics: "When a custom event is fired the event is put on the queue and will be handled in the main thread when Fusion is idle"; it "Returns true if the event was successfully added to the event queue" — queuing, not execution ([Application.fireCustomEvent Method](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Application_fireCustomEvent.htm) — fetched 2026-06-11). Also: "Python runs within the Fusion process and also runs in the main Fusion thread. Because of this, when your program is running [most] of Fusion appears frozen" ([Python Specific Issues](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — fetched 2026-06-11). Documented failure mode for violations: crashes (the messageBox quote above).

### 6. Bundling third-party Python packages

Officially supported: **vendoring**. "The Python provided with Fusion includes only the core modules that come with a standard installation of Python. However, you can use other modules by making them available to your Python program. Instead of installing or adding the module to sys.path we recommend that you have a local copy of the module for your script… you install the Python module in the same directory as your script or a subdirectory," imported with relative syntax, e.g. `from .Modules import xlrd` ([Python Specific Issues](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — fetched 2026-06-11). The phrasing implicitly acknowledges `sys.path` manipulation works but recommends against it. Running pip against Fusion's embedded interpreter is **not documented anywhere official** — that is a community workaround only [community practice, unverified here]. Distribution caveat: `.pyc`-only distribution is fragile — ".pyc files are tied to the version of Python used to create them… This change does not affect any add-ins where the .py source code is available," and Autodesk shipped in-product warnings/errors for stale `.pyc` add-ins during the 3.7→3.9 transition ([forum announcement](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/important-python-version-change-information/td-p/10929351)).

### 7. Debugging story

VS Code is the official toolchain: "Fusion uses Visual Studio Code (VS Code) as the development environment"; Fusion offers to install it on first edit, and "To debug Python code, VS Code requires the optional 'ms-python.python' extension. This extension is automatically installed the first time Fusion opens VS Code" ([Creating a Script or Add-In](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/WritingDebugging_UM.htm); [Python Specific Issues](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — both fetched 2026-06-11). Workflow: from the Scripts and Add-Ins dialog, "Next to the Run button, click the dropdown arrow, then click Debug" ([Manage scripts and add-ins](https://help.autodesk.com/cloudhelp/ENU/Fusion-Model/files/SLD-MANAGE-SCRIPTS-ADD-INS.htm) — fetched 2026-06-11), or start from VS Code via F5/Run menu; "Fusion will start executing your script but will stop execution at the first break point it hits"; for add-ins, "Fusion loads the add-in and calls the run method, just like it does for scripts" ([Python Specific Issues]). The docs do **not** document the attach mechanism (debugpy, port) — community sources describe a debugpy attach under the hood [unverified]. No official logging framework is documented in the fetched pages; the threads page suggests `ctypes.windll.user32.MessageBoxW` for thread-safe debug messages on Windows.

### 8. Auto-load and consent gating

Yes — add-ins can run at startup. The `runOnStartup` manifest field plus the dialog setting: "Run on Startup – This setting is add-in specific and indicates if the add-in should be run automatically when Fusion is started. Most add-ins will want to take advantage of this capability so the commands they define will be available to the user as soon as Fusion starts" ([Creating a Script or Add-In] — fetched 2026-06-11). The add-in can detect the mode via `IsApplicationStartup` in `run(context)`. **Consent/trust gating: none documented.** The Scripts and Add-Ins user-manual page and the API manual pages fetched describe no warning, signing check, or trust prompt before an add-in executes ([Manage scripts and add-ins] — explicit absence, fetched 2026-06-11). The only documented gates are upstream: App Store review for store-distributed apps, and the macOS Mac-App-Store sandbox restricting where bundles may load from. Whether any newer Fusion build adds a first-run prompt for manually installed add-ins: could not determine from official docs [unverified].

## Decision implications for adze-fusion

- **Modern runtime, stale assumptions purged:** Python 3.14 means current typing, asyncio, and wide wheel compatibility — the vault must correct any 3.7-era assumption. But Autodesk upgrades the interpreter roughly on its own cadence; ship `.py` source (never `.pyc`-only) and pure-Python vendored deps to survive version bumps. Compiled-extension deps (numpy etc.) are version-locked risk.
- **Architecture is dictated by the main-thread rule:** any localhost listener (the dominant MCP-bridge pattern) must run on a worker thread and marshal every API call through `fireCustomEvent`, executing "when Fusion is idle" — i.e., latency is event-queue-bound, not socket-bound.
- **Two distribution geometries:** dev/manual installs live in `API/AddIns` (flat folder); App Store installs live in `ApplicationPlugins` (.bundle + PackageContents.xml) and Autodesk builds the installer. Plan for both layouts plus the Mac App Store container path from day one.
- **Zero-friction persistence:** `runOnStartup` gives an always-present agent surface with no user trust prompt — good for UX, and it makes the publisher-guideline load-time budget the real constraint.

## Open questions

- Actual on-disk folder name on current installs: `Autodesk Fusion` vs legacy `Autodesk Fusion 360` (docs renamed; disk reality needs an [empirical] check).
- Verbatim current `autodeskProduct` manifest value ("Fusion360" vs "Fusion").
- Debugger transport details (debugpy? port? attach config) — undocumented officially.
- Whether App Store installers require admin elevation (ADN FAQ claim vs per-user folder).
- Full `PackageContents.xml` schema for Fusion (the ADN developer-info page was unreachable: ECONNREFUSED).
- GIL vs free-threaded 3.14 build in Fusion.

## Sources

- [2026 Release Notes — Fusion Help](https://help.autodesk.com/cloudhelp/ENU/Fusion-ReleaseNotes/files/FIC-REL-NOTES-INC.htm) — fetched 2026-06-11 — "Updated the embedded Python version from 3.12 to 3.14" (Build v.2606.1.22, January 21, 2026).
- [Creating a Script or Add-In — Fusion Help](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/WritingDebugging_UM.htm) — fetched 2026-06-11 — lifecycle, manifest fields, default folders, discovery rule, Run on Startup, VS Code.
- [Python Specific Issues — Fusion Help](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/PythonSpecific_UM.htm) — fetched 2026-06-11 — vendoring modules, main-thread execution, VS Code debugging, ms-python extension.
- [Working in a Seperate Thread — Fusion Help](https://help.autodesk.com/view/fusion360/ENU/?guid=GUID-F9FD4A6D-C59F-4176-9003-CE04F7558CCC) — fetched 2026-06-11 (via Tavily; WebFetch 404) — main-thread rule, crash warning, custom-event pattern.
- [Application.fireCustomEvent Method — Fusion Help](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/Application_fireCustomEvent.htm) — fetched 2026-06-11 — queue/idle semantics, return value.
- [Manage scripts and add-ins — Fusion Help](https://help.autodesk.com/cloudhelp/ENU/Fusion-Model/files/SLD-MANAGE-SCRIPTS-ADD-INS.htm) — fetched 2026-06-11 — dialog Run/Stop/Debug; no trust prompts documented.
- [How to install an add-in or script in Autodesk Fusion — Autodesk Support (Apr 10, 2026)](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-to-install-an-ADD-IN-and-Script-in-Fusion-360.html) — fetched 2026-06-11 (via Tavily; WebFetch 403) — install paths, App Store installer flow, manual copy.
- [Fusion app paths — Autodesk Developer Blog](https://blog.autodesk.io/fusion-app-paths/) — fetched 2026-06-11 — ApplicationPlugins paths incl. Mac App Store container; sandbox rationale.
- [Fusion publishing guidelines — APS Publisher Center](https://aps.autodesk.com/app-store/publisher-center/fusion-360) — fetched 2026-06-11 — .bundle + PackageContents.xml + Contents; Autodesk-built installer; per-user installs; load-time budget.
- [Important Python Version Change Information — Autodesk Community forum (official Autodesk post, Feb 2022)](https://forums.autodesk.com/t5/fusion-api-and-scripts-forum/important-python-version-change-information/td-p/10929351) — fetched 2026-06-11 — 3.7→3.9.7 transition; .pyc breakage mechanics and in-product warnings.
- [What's New — Fusion API](https://help.autodesk.com/cloudhelp/ENU/Fusion-360-API/files/WhatsNew.htm) — fetched 2026-06-11 — May 2026 Electronics API Python support; no version entries on this page itself.
- Unreachable: [Fusion 360 Developer Info — ADN App Store page](https://app.upskill-dev.autodesk.com/developer-network/app-store/fusion-360) — ECONNREFUSED 2026-06-11; PackageContents.xml schema and admin-privileges claim therefore rest on search snippets, marked (via search).
