# Adversarial review — W1-1 (Fusion Python add-in API: runtime, lifecycle, packaging, install path)

**Reviewer:** P0 adversarial sub-agent
**Date:** 2026-06-11
**Target:** `research/findings/W1-1-fusion-python-runtime.md`

## Verdict at a glance

**accept-with-corrections (A-).** All 8 numbered questions answered. 7 citation spot-checks run; 6 pass verbatim, 1 reveals a real misquote on the install-path folder name (the cited doc page is internally inconsistent, and the finding reports the conflict backwards for the AddIns path). One documented discovery directory (`FusionAddins`) was missed entirely, which makes the word "only" in the discovery claim wrong. The headline claim (embedded Python 3.14) survived an active refutation attempt. All 6 self-reported [unverified] claims are genuinely marked in the text. No fabricated or misrepresented sources.

## Spot-checks performed

1. **2026 Release Notes** (`help.autodesk.com/.../FIC-REL-NOTES-INC.htm`) — PASS. "Updated the embedded Python version from 3.12 to 3.14" confirmed under Build v.2606.1.22 (January 21, 2026), API section. Haas/Renishaw Feb 4, 2026 note also confirmed verbatim.
2. **Creating a Script or Add-In** (`WritingDebugging_UM.htm`) — PARTIAL FAIL on paths (see Dangerous claims), PASS on everything else: run/stop semantics, `IsApplicationStartup`/`IsApplicationClosing`, manifest field list, "any location on the machine… green '+' icon" discovery quote, VS Code, Run on Startup — all confirmed. Bonus: the page states `autodeskProduct` "will always have the value 'Fusion'", partially resolving the finding's hedge.
3. **Python Specific Issues** (`PythonSpecific_UM.htm`) — PASS. No version number stated on page (confirmed absence); vendoring quote incl. `from .Modules import xlrd`; "runs in the main Fusion thread… appears frozen"; ms-python.python auto-install — all verbatim.
4. **Application.fireCustomEvent** — PASS. "put on the queue and will be handled in the main thread when Fusion is idle" and the returns-true-means-queued-not-handled semantics confirmed verbatim.
5. **Fusion app paths — Autodesk Developer Blog** — PASS. All three ApplicationPlugins paths (Windows, Mac web, Mac App Store container `com.autodesk.mas.fusion360`), "looking for add-in bundles to load", and the MAS sandbox quote confirmed verbatim.
6. **Feb 2022 forum announcement** (3.7→3.9.7) — WebFetch 403; corroborated via search: 3.9.7 upgrade, "Add-ins compiled with the current version of Python (3.7) will not be compatible", .pyc version-tie mechanics, in-product warnings all match. PASS (indirect; staff authorship not independently confirmed).
7. **APS publisher guidelines (Fusion)** — PASS. "Keep your load time under 0.005 seconds", `.bundle` + `PackageContents.xml` + `Contents` subfolder, Autodesk-built installer recommendation, "Apps installed in the per-user folder are only available to that user" — all confirmed verbatim.

**Refutation attempt on the most load-bearing claim (Python 3.14):** independent search surfaced (a) the official release notes, (b) a Jan 2026 Autodesk forum thread "Fusion 360 Python 3.14 update breaks cryptography/Google auth imports", (c) the January 2026 Product Update blog. Claim stands. Refutation failed.

**[unverified] marker audit:** all 6 researcher-self-reported claims (GIL build, autodeskProduct verbatim, on-disk folder name, App Store admin elevation, debugpy attach, pip-into-embedded-runtime) confirmed marked in the file text (lines 20, 30, 42/84, 50, 69, 65 respectively).

## Dangerous claims

1. **"Verbatim from the current API user manual… `%appdata%\Autodesk\Autodesk Fusion\API\AddIns`" + "current official pages say `Autodesk Fusion`, not `Autodesk Fusion 360`" (Section 3, Summary).** The cited page actually renders the **AddIns** paths as `…\Autodesk\Autodesk Fusion 360\API\AddIns` and only the **Scripts** paths as `…\Autodesk\Autodesk Fusion\API\Scripts`. The doc is internally inconsistent; the finding's "current vs legacy docs" framing is wrong — both names appear on the same current page. The finding self-flagged the uncertainty as an open question, so this is fixable, not fabrication — but the install path is load-bearing for any prototype/installer work and must not be promoted as quoted.
2. **"Fusion auto-discovers them only in one per-user `API/AddIns` directory per OS" (Summary).** The same cited page also documents `%appdata%\Autodesk\FusionAddins` (Windows) and `~/Library/Application Support/Autodesk/FusionAddins` (Mac) as add-in locations, plus the ApplicationPlugins bundle dirs. "Only … one … per OS" is false as written.

## Corrections

1. Section 3 / Summary: replace the "current docs say `Autodesk Fusion`" claim with: the cited page is internally inconsistent — Scripts lines render `Autodesk Fusion`, AddIns lines render `Autodesk Fusion 360`. The [empirical] on-disk check remains required; do not promote either name as authoritative.
2. Section 3 / Summary: add the documented `FusionAddins` discovery directories and delete "only in one per-user API/AddIns directory per OS".
3. Section 2: upgrade the `autodeskProduct` hedge — the current doc rendering states the value is `'Fusion'` ("This property will always have the value 'Fusion'"); keep [unverified] only for what shipping manifests in the wild use (legacy `"Fusion360"` is common).
4. Section 3 (minor): the Preferences quote — the page rendering fetched says "The default path can be redefined by setting a user preference," not the quoted "'General' -> 'API section' in the Preferences command." Re-cite or soften to paraphrase.

## Missed topics

- The `FusionAddins` directory pair documented on the very page cited (also a Dangerous claim above).
- Known 3.14-transition breakage in the wild: Jan 2026 forum reports of add-ins failing on native-module packages (cryptography / Google auth) after the 3.12→3.14 bump — directly strengthens the finding's own "compiled-extension deps are version-locked risk" implication; worth one sourced line.
- (Nit, non-blocking) Source title typo carried from Autodesk: "Working in a Seperate Thread" — sic in original, fine as cited.
