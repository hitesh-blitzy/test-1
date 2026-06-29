# Blitzy Project Guide
## Integrations Redesign & Folder-Level Team Access — Behavioral/Visual Prototype

---

## 1. Executive Summary

### 1.1 Project Overview

This project redesigns the Blitzy **Workspace → Settings → Integrations** surface and introduces a **folder-level team-access model** for source-control (SCM) and design providers, delivered as an authoritative behavioral-and-visual prototype across two static HTML files. It targets Blitzy workspace administrators (who connect providers and push folder-scoped team access) and team members (read-only). The prototype replaces the legacy flat provider grid with a two-pane shell, groups provider variants under company cards with **per-variant** status ("club the look, split the access"), and adds a folder-first sharing modal with push-only, top-level-folder grants and automatic downward inheritance. Its technical scope is the complete front-end behavior and visual system that the downstream production codebase must satisfy.

### 1.2 Completion Status

The project is **84.1% complete** measured against AAP-scoped work (the in-repository prototype deliverable plus its immediate path-to-production). The prototype implementation is complete and validated; the remaining hours are path-to-production decisions, sign-off, and quality passes — not code defects.

```mermaid
%%{init: {"theme":"base","themeVariables":{"pie1":"#5B39F3","pie2":"#FFFFFF","pieStrokeColor":"#5B39F3","pieStrokeWidth":"2px","pieOuterStrokeColor":"#5B39F3","pieOuterStrokeWidth":"2px","pieTitleTextColor":"#5B39F3","pieLegendTextColor":"#1c1c22","pieSectionTextColor":"#1c1c22"}}}%%
pie title Completion Status — 84.1% Complete (AAP-scoped)
    "Completed Work (AI) 106h" : 106
    "Remaining Work 20h" : 20
```

| Metric | Hours |
|--------|-------|
| **Total Hours** | **126** |
| Completed Hours (AI + Manual) | 106 |
| &nbsp;&nbsp;• AI (autonomous) | 106 |
| &nbsp;&nbsp;• Manual (human) | 0 |
| **Remaining Hours** | **20** |
| **Percent Complete** | **84.1%** |

> Legend — **Completed = Dark Blue `#5B39F3`**, **Remaining = White `#FFFFFF`**.

### 1.3 Key Accomplishments

- ✅ **All twelve features (F-001…F-012)** implemented and validated in the primary prototype.
- ✅ **Two-pane data-driven catalogue** — 200px category rail (SCM/Design) built from the `DATA` registry; company cards stacked with per-variant sub-cards; empty-state and "More to come" hints.
- ✅ **Per-variant state machine** — five states (`none/connecting/connected/failed/soon`) resolved independently per sub-card; the `mixed` preset proves independence; dead `rollup()` removed so no company-level status can render.
- ✅ **Connect flows** — OAuth redirect dialog (cloud) with `none → connecting → connected` at 1100 ms, and a self-hosted credentials form (URL/App ID/Secret, all required, Connect disabled until valid) with **inline URL syntax + reachability** errors.
- ✅ **Folder-first sharing modal (largest effort)** — stable-ID anchored grant tuples `{connectionId, folderStableId, folderPathSnapshot, teamId}`, read-time inheritance, direct-vs-inherited display with lock + provenance, inline redundant-grant blocking, per-connection isolation.
- ✅ **Role-based dispatch** — Super Admin controls vs read-only Team Member, with an explicit server-side re-authorization (defense-in-depth) spec documented in-file.
- ✅ **Secondary file reconciled** — canonical tokens + Inter, Tabler Icons @3.7.0, out-of-scope UI removed, inheritance affordances added.
- ✅ **Hardening** — SRI + crossorigin, ARIA dialog/menu, focus trap + return, `prefers-reduced-motion`, password fields, icon labels, `esc()` XSS escaping.
- ✅ **Autonomous validation** — 61 behavioral assertions pass; zero console errors in Chrome; responsive at 375/768/1280/1440; `node --check` + `htmlhint` clean; Figma visual fidelity confirmed (0 blocker/major/minor).

### 1.4 Critical Unresolved Issues

There are **no blocking code defects**. The validator confirmed zero compilation errors and zero failing tests in the prototype. The items below are decisions/preconditions that gate downstream production — not failures in the delivered artifact.

| Issue | Impact | Owner | ETA |
|-------|--------|-------|-----|
| Stable-ID folder-persistence precondition (F-010) unconfirmed | Determines whether the production grant model is a new addition or an alignment; blocks finalizing the downstream contract | Backend / Architecture | 3h |
| Team-tier gating undecided (AAP open question) | Affects downstream scope (does the Team tier receive folder grants?) | Product | 1.5h |
| Existing-share → folder-model migration undefined | Affects rollout of the new model over legacy whole-integration shares | Product / Backend | 2h |
| Tertiary-text contrast (`--ter` `#999`, ~2.6–2.85:1) below WCAG AA | Accessibility; design-authoritative per Figma/AAP but flagged for review | Design | within 2.5h a11y task |

### 1.5 Access Issues

**No access issues identified.** The repository, both source files, the branch (`blitzy-80043409-…`), and the CDN dependencies (Tabler Icons webfont @3.7.0 and Google Fonts Inter — both returned HTTP 200) were all accessible during validation. There are no service credentials, private registries, or third-party API keys required for the prototype.

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|-----------------|----------------|-------------------|-------------------|-------|
| Repository / branch `blitzy-80043409-…` | Read/Write | None | ✅ Accessible | — |
| jsDelivr CDN (Tabler Icons @3.7.0) | HTTPS GET | None (HTTP 200) | ✅ Accessible | — |
| Google Fonts (Inter) | HTTPS GET | None (HTTP 200) | ✅ Accessible | — |

### 1.6 Recommended Next Steps

1. **[High]** Confirm the stable-ID folder-persistence precondition (F-010) in existing production code before finalizing the downstream grant contract. *(3h)*
2. **[High]** Obtain stakeholder + design sign-off of the prototype against the Figma frames (Bitbucket `52109:53707` / GitLab `35972:2977`, Section F card-state board). *(3h)*
3. **[Medium]** Produce the downstream handoff package: prototype→production mapping, token map, and component inventory for the React/TypeScript team. *(4h)*
4. **[Medium]** Run a formal automated accessibility audit (axe-core/Lighthouse) and resolve the flagged tertiary-text contrast with design. *(2.5h)*
5. **[Low]** Cross-browser smoke beyond Chrome (Firefox + Safari/WebKit) and optional preview hosting for review. *(3.5h)*

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

Every completed component traces to a specific AAP requirement (feature ID and/or work group). Total = **106 hours**, matching Completed Hours in §1.2.

| Component | Hours | Description |
|-----------|-------|-------------|
| Page shell & catalogue *(G1 / F-001)* | 8 | Two-pane `200px minmax(0,1fr)` layout, ten settings tabs (incl. added "Web search"), data-driven category rail (`renderRail()` from `Object.keys(DATA)` + `CATS`), empty state, responsive media queries. |
| Cards, badges, buttons & state machine *(G2 / F-002, F-003)* | 13 | Company + per-variant sub-cards, ADO "AZ" mark, 4-state badge system, full button taxonomy, `st()/badge()/actions()` state machine, `PRESETS`, spinner with reduced-motion, layout-shift prevention, dead `rollup()` + `.rollup` removed. |
| Provider/variant data model *(F-004)* | 5 | `DATA` (GitHub/GitLab/Azure DevOps/Bitbucket + Figma; 7 sub-cards), `FORMS`, `CATS`, `FOLDERS` registries with `connect` descriptors. |
| Connect dialogs *(G3 / F-005, F-006)* | 11 | `oauthModal()` ("Authorize on {provider}", 1100 ms transition) + `formModal()` (all-required, Connect disabled until valid) with inline URL **syntax** and **reachability** errors (`validUrl`/`reachable` RFC 2606/6761), semantic `<form>` wrapper. |
| Role dispatch, 3-dot menu & confirmations *(G4 / F-007, F-008)* | 8 | `actions()` role×status dispatch (admin vs read-only member), `openMenu()` with keyboard nav and exact row set, verbatim Disconnect/Revoke `confirmD()` copy, refresh/disconnect/revoke transitions. |
| Folder-first sharing modal *(G5 / F-009, F-010)* | 18 | Rebuilt `shareModal()`: stable-ID tuple grants, read-time inheritance (`ancestorsOf`/`inheritedFor`/`pathOf`), direct-vs-inherited UI with lock + provenance, inline redundant-grant blocking, per-connection isolation, case-insensitive team search. |
| Demo control strip *(F-011)* | 3 | Prototype-only state/role scenario toolbar + live status log + scenario/role wiring, with explicit strip-boundary comments. |
| Design-token system *(F-012)* | 4 | Canonical 16-token `:root` block as single source of truth + Figma token mapping/reconciliation (~75 `var()` references). |
| Accessibility & security hardening *(G8)* | 8 | ARIA dialog/menu, focus trap + return, SRI + crossorigin, `type="password"` secrets, `prefers-reduced-motion`, icon-only button labels, `esc()` XSS escaping (CWE-79), data-URI favicon. |
| Secondary file reconciliation *(G6)* | 11 | `folder-sharing-prototype-v2.html`: token consolidation to canonical + Inter, Tabler @3.7.0, removed out-of-scope UI (mode toggle, access-level pills, repo/branch levels, carve-out), added inheritance affordances, clean delegated event wiring. |
| Downstream forward-reference documentation *(G7)* | 2 | In-file documentation of production targets (`integrations.tsx`, `SvcType`, `IntegrationTeamShareRequest`, `bulkUpdateIntegrationTeamAccess`, `GITLAB_SELF_HOSTED`, `TreeNode`, pickers) + defense-in-depth server-side authorization spec. |
| Autonomous validation & visual fidelity *(Blitzy QA)* | 15 | 61-assertion behavioral harness, full Chrome runtime exercise, responsive validation (4 breakpoints), `node --check`/`htmlhint`, Figma `compare_screenshot_with_figma` vs 2 boards + 4 fidelity fixes, commit. |
| **Total** | **106** | |

### 2.2 Remaining Work Detail

Each remaining item traces to an AAP open question (§0.9.3), a path-to-production need, or an identified risk. Total = **20 hours**, matching Remaining Hours in §1.2 and the Section 7 pie chart.

| Category | Hours | Priority |
|----------|-------|----------|
| Architecture precondition — confirm stable-ID folder persistence (F-010) | 3 | High |
| Design review & sign-off — prototype vs Figma frames | 3 | High |
| Production handoff package — prototype→production mapping, token map, component inventory | 4 | Medium |
| Accessibility & quality — formal axe/Lighthouse audit + `--ter` contrast triage | 2.5 | Medium |
| Product decision — team-tier gating (AAP open question) | 1.5 | Medium |
| Migration strategy — existing whole-integration shares → folder model | 2 | Medium |
| Cross-browser QA — Firefox + Safari/WebKit smoke of all flows | 2 | Low |
| Preview deployment — optional static hosting for review | 1.5 | Low |
| Spec reconciliation — `rollup()` framing with spec owner | 0.5 | Low |
| **Total** | **20** | |

### 2.3 Hours Reconciliation

| Lens | Breakdown | Sum |
|------|-----------|-----|
| Completed (§2.1) + Remaining (§2.2) | 106 + 20 | **126** (= Total in §1.2) ✓ |
| Remaining by priority | High 6 + Medium 10 + Low 4 | 20 ✓ |
| Remaining by type | AAP open questions 7 + Path-to-production 8.5 + Quality 4.5 | 20 ✓ |

---

## 3. Test Results

The repository contains **no formal unit-test suite** (a static HTML prototype with no build system); there were no repository tests to run. All results below originate exclusively from **Blitzy's autonomous validation logs** for this project.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---------------|-----------|-------------|--------|--------|------------|-------|
| Behavioral assertions | Custom Node `vm` + DOM-stub harness | 61 | 61 | 0 | n/a (function-level) | `esc()` XSS-escaping; `reachable()` host logic; `find`/`findProv`; `badge`; `st`/`setScenario` across zero/mixed/ideal/connecting; `actions()` role dispatch; secondary `parentOf`/`ancestors` read-time inheritance, redundant-grant blocking, per-connection grant isolation. |
| Runtime (browser) | Chrome via `file://` | 2 files | 2 | 0 | n/a | Both files render and run with **zero console errors**; all flows exercised. |
| Responsive | Chrome DevTools (375/768/1280/1440) | 4 breakpoints | 4 | 0 | n/a | Zero horizontal overflow, zero console errors at every breakpoint. |
| Static syntax check | `node --check` | 2 files | 2 | 0 | n/a | Inline JS of both files valid. |
| HTML lint | `htmlhint` | 2 files | 2 | 0 | n/a | 0 errors both files. HTML5 structural conformance pass. |
| Visual fidelity | `compare_screenshot_with_figma` | 87 props (connect dialog) | 50 match | 0 actionable | n/a | 35 discrepancies all graded INFO (documented AAP overrides/page-chrome); **0 blocker/major/minor**. 4 fidelity fixes applied and re-verified. |

> **Integrity note:** Every test above is sourced from Blitzy's autonomous test/validation execution. No external or fabricated results are included.

---

## 4. Runtime Validation & UI Verification

Status legend: ✅ Operational | ⚠ Partial | ❌ Failing

**Primary prototype — `blitzy-integrations-page.html`**
- ✅ Page renders in Chrome with zero console errors
- ✅ Data-driven category rail (SCM active / Design), empty-state, "More to come" hint
- ✅ Scenario presets: Zero / Connected (mixed) / Ideal (all) / Connecting
- ✅ Role toggle: Super Admin (full controls) vs Team Member (read-only labels)
- ✅ OAuth connect: redirect dialog → `connecting` → `connected` (~1100 ms)
- ✅ Self-hosted form connect: valid submit, invalid-URL inline error, unreachable-URL inline error
- ✅ 3-dot menu: Refresh connection / Share folder access / Disconnect / Revoke access
- ✅ Destructive confirmations: distinct Disconnect (reversible) vs Revoke access (irreversible), verbatim copy
- ✅ Folder-first share modal: add / remove (direct) / inherited (locked, with provenance) / redundant-grant block
- ✅ Keyboard: Esc + Tab focus trap and focus return; arrow-key menu navigation
- ✅ ARIA dialog (`role="dialog"`, `aria-modal`, `aria-labelledby`) + focus management
- ✅ Responsive at 375 / 768 / 1280 / 1440 (two-pane ≥1280; single-column reflow + title-ellipsis at 375)

**Secondary prototype — `folder-sharing-prototype-v2.html`**
- ✅ List view (direct grants removable; inherited grants locked with provenance)
- ✅ Picker view: folder select → team toggle → redundant block → Add / Save & add another → Done summary
- ✅ Top-level folders only (workspace + project); no repo/branch levels, no access-level pills, no carve-out toggle

**API / dependency integration**
- ✅ CDN assets reachable: Tabler Icons webfont @3.7.0 (HTTP 200), Google Fonts Inter (HTTP 200)
- ✅ SRI integrity + crossorigin present and matching on both files

**Partial / advisory**
- ⚠ Cross-browser validation beyond Chrome not yet performed (remaining Low-priority task)
- ⚠ Tertiary-text contrast (`--ter` `#999`) below WCAG AA for small text — design-authoritative per Figma/AAP, flagged for review

No failing (❌) runtime items.

---

## 5. Compliance & Quality Review

AAP deliverables and key constraints cross-mapped to Blitzy quality/compliance benchmarks.

| Deliverable / Benchmark | Status | Progress | Notes |
|-------------------------|--------|----------|-------|
| F-001 Data-driven category navigation | ✅ Pass | 100% | `renderRail()` from `DATA` keys + `CATS`; empty state. |
| F-002 Provider company cards (no status rollup) | ✅ Pass | 100% | `render()`; ADO "AZ" mark; `rollup()` removed. |
| F-003 Per-variant sub-cards, independent status | ✅ Pass | 100% | `st()` per sub-card; `mixed` preset proves independence. |
| F-004 Connection variant definitions | ✅ Pass | 100% | 7 sub-cards; `connect{kind:'oauth'|'form'}`. |
| F-005 OAuth connect dialog | ✅ Pass | 100% | `oauthModal()`; 1100 ms transition. |
| F-006 Credentials form connect dialog | ✅ Pass | 100% | `FORMS`-driven; all-required; disabled-until-valid; inline URL error. |
| F-007 Role-based action dispatch | ✅ Pass | 100% | `actions()` admin vs member; server-side re-auth documented. |
| F-008 3-dot management menu | ✅ Pass | 100% | Exact row set; admin-only on connected/failed. |
| F-009 Folder-level team sharing dialog | ✅ Pass | 100% | Direct-vs-inherited; redundant-grant block; per-connection. |
| F-010 Stable-ID anchored grant persistence | ✅ Pass (prototype) | 100% | Tuple model + read-time inheritance; production precondition is an open question (§6 I2). |
| F-011 Demo control strip (prototype-only) | ✅ Pass | 100% | Strip-boundary comments; excluded from production build. |
| F-012 Design token reuse | ✅ Pass | 100% | Canonical 16-token `:root` in both files. |
| Constraint — canonical tokens verbatim | ✅ Pass | ~95%+ | Documented one-off provider-brand literals only. |
| Constraint — never label action "Uninstall" | ✅ Pass | 100% | Distinct Disconnect / Revoke access. |
| Constraint — push-only, top-level folders, no carve-outs | ✅ Pass | 100% | Enforced in both files. |
| Constraint — per-connection access isolation | ✅ Pass | 100% | Grants keyed by connection id. |
| Hardening — SRI, ARIA, focus trap, reduced-motion, password, XSS | ✅ Pass | 100% | Verified in both files. |
| Quality — automated accessibility audit | ⚠ Partial | Pending | Manual ARIA/keyboard validated; formal axe/Lighthouse pass remaining. |
| Quality — cross-browser verification | ⚠ Partial | Pending | Chrome validated; Firefox/WebKit remaining. |
| Accessibility — tertiary-text contrast | ⚠ Flagged | Design decision | `--ter` `#999` below AA; design-authoritative, flagged in-file. |

**Fixes applied during autonomous validation:** (1) kebab trigger chrome — removed bordered box, recolored dots to `#999999`; (2) scrim tint `rgba(0,0,0,.4)` → `rgba(0,0,0,.2)`; (3–4) "Learn how ↗" help-link focus — keyboard-only focus ring + initial focus skips the help link to land on the URL field.

**Outstanding (non-blocking):** formal automated a11y audit, cross-browser smoke, and the tertiary-contrast design decision.

---

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|------|----------|----------|-------------|------------|--------|
| T1 — CDN runtime dependency (Tabler + Inter required at render time) | Technical | Medium | Medium | Self-host fonts/icons for offline review; downstream bundles them | Open (by design) |
| T2 — No committed automated regression suite | Technical | Low | Medium | Commit the 61-assertion harness or add Playwright smoke if prototype keeps evolving | Open (accepted) |
| T3 — Simulated behavior (timers, `reachable()` heuristic) not real OAuth/network | Technical | Low | Low | Documented in-file as forward-reference; downstream replaces with real probes | Mitigated |
| S1 — Client-side role gating is not a security boundary | Security | High | Low | Header spec mandates server-side re-authorization (HTTP 403) for every privileged mutation | Documented / downstream |
| S2 — Tertiary-text contrast (`--ter` `#999`) below WCAG AA | Security (a11y) | Medium | Medium | Flagged in-file for designer review; downstream may add accessible tertiary token | Open (design) |
| S3 — CDN supply-chain | Security | Low | Low | SRI integrity hashes + crossorigin on both files; self-host for production | Mitigated |
| S4 — Cross-site scripting (CWE-79) | Security | Low | Low | `esc()` escapes all dynamic leaf values at the render boundary in both files | Mitigated |
| O1 — Single-browser validation (Chrome only) | Operational | Low | Medium | Cross-browser smoke (remaining task) | Open |
| O2 — Demo strip must be removed for production | Operational | Low | Low | Strip-boundary comments; downstream excludes the block + `.demo` CSS | Documented |
| O3 — No monitoring/logging/health checks | Operational | Low | Low | Not applicable to a static prototype; downstream concern | Out of scope |
| I1 — Downstream production targets unimplemented | Integration | High | Certain (pending) | Prototype authoritatively specifies behavior; AAP documents all targets | Open / out of scope |
| I2 — Stable-ID persistence precondition unconfirmed (F-010) | Integration | Medium | Medium | Confirm in existing production before finalizing model (remaining task) | Open (AAP Q) |
| I3 — Team-tier gating & share migration undefined | Integration | Medium | Medium | Resolve with product (remaining tasks) | Open (AAP Q) |

---

## 7. Visual Project Status

**Overall progress (hours)** — Completed = Dark Blue `#5B39F3`, Remaining = White `#FFFFFF`.

```mermaid
%%{init: {"theme":"base","themeVariables":{"pie1":"#5B39F3","pie2":"#FFFFFF","pieStrokeColor":"#5B39F3","pieStrokeWidth":"2px","pieOuterStrokeColor":"#5B39F3","pieOuterStrokeWidth":"2px","pieTitleTextColor":"#5B39F3","pieLegendTextColor":"#1c1c22","pieSectionTextColor":"#1c1c22"}}}%%
pie title Project Hours Breakdown (Total 126h)
    "Completed Work" : 106
    "Remaining Work" : 20
```

**Remaining work by priority (20h)** — accent palette.

```mermaid
%%{init: {"theme":"base","themeVariables":{"pie1":"#5B39F3","pie2":"#B23AF2","pie3":"#A8FDD9","pieStrokeColor":"#1c1c22","pieStrokeWidth":"1px","pieTitleTextColor":"#5B39F3","pieLegendTextColor":"#1c1c22","pieSectionTextColor":"#1c1c22"}}}%%
pie title Remaining Work by Priority
    "High" : 6
    "Medium" : 10
    "Low" : 4
```

**Remaining hours by category (from §2.2):**

| Category | Hours |
|----------|-------|
| Production handoff package | 4.0 |
| Architecture precondition (stable-ID) | 3.0 |
| Design review & sign-off | 3.0 |
| Accessibility & quality audit | 2.5 |
| Migration strategy | 2.0 |
| Cross-browser QA | 2.0 |
| Team-tier gating decision | 1.5 |
| Preview deployment | 1.5 |
| Spec reconciliation (`rollup()`) | 0.5 |
| **Total** | **20.0** |

> **Integrity check:** "Remaining Work" = **20h** in the pie above equals Remaining Hours in §1.2 and the sum of the §2.2 Hours column.

---

## 8. Summary & Recommendations

**Achievements.** The Integrations redesign and folder-level team-access model are delivered as a complete, validated behavioral/visual prototype. All twelve features (F-001…F-012) are implemented across the two HTML files, every special constraint is honored ("club the look, split the access"; Disconnect vs Revoke; push-only; top-level-folder inheritance; per-connection isolation; canonical tokens), and cross-cutting hardening (SRI, ARIA, focus management, reduced-motion, password fields, XSS escaping) is in place. Blitzy's autonomous validation confirms zero console errors, 61/61 behavioral assertions passing, clean responsive behavior at four breakpoints, and Figma visual fidelity with zero blocker/major/minor discrepancies.

**Remaining gaps.** The project is **84.1% complete (106 of 126 AAP-scoped hours)**. The remaining **20 hours** contain **no code defects** — they are path-to-production activities: confirming the stable-ID persistence precondition (F-010), stakeholder/design sign-off, the downstream handoff package, two open product decisions (team-tier gating, share migration), a formal accessibility audit plus the tertiary-contrast design decision, cross-browser smoke, and optional preview hosting.

**Critical path to production.** (1) Confirm the F-010 stable-ID precondition → (2) secure design/stakeholder sign-off → (3) deliver the handoff package to the downstream team. These three items (10h) unblock the production implementation.

> **Important scope clarification.** This repository's deliverable is the **prototype**. The downstream **React/TypeScript production implementation** — `src/panel/workspace/settings/integrations.tsx`, the `SvcType` enum extension, `IntegrationTeamShareRequest` / `bulkUpdateIntegrationTeamAccess`, the `GITLAB_SELF_HOSTED` adapter, `TreeNode` reuse, and project-picker read-side filtering — is **explicitly out of scope** (forward-references) and is **not** included in the 84.1% figure. It is a separate, substantial engineering effort that the prototype now authoritatively specifies. The 84.1% should be read as "the prototype deliverable and its immediate path-to-production," not "the entire feature in production."

**Production readiness assessment.** As a **prototype deliverable**, the artifact is production-ready: validated, defect-free, and faithfully aligned to the Blitzy design system and Figma source of truth. As a **production feature**, it is a specification awaiting the downstream build. Recommendation: **approve the prototype, resolve the open questions, and hand off to the production team.**

| Success Metric | Target | Status |
|----------------|--------|--------|
| Features implemented (F-001…F-012) | 12 / 12 | ✅ 12 / 12 |
| Behavioral assertions passing | 100% | ✅ 61 / 61 |
| Console errors at runtime | 0 | ✅ 0 |
| Responsive breakpoints clean | 4 / 4 | ✅ 4 / 4 |
| Blocker/major/minor visual discrepancies | 0 | ✅ 0 |
| AAP-scoped completion | — | 84.1% |

---

## 9. Development Guide

The deliverable is **static HTML with CDN-only dependencies** — there is **no build, install, or compile step**. All commands below were executed and verified in the validation environment (Node v20.20.2, Python 3.13.13).

### 9.1 System Prerequisites

- A modern browser: Chrome/Chromium (validated), Firefox, or Safari/WebKit.
- **Internet access at render time** for the Tabler Icons webfont and Google Fonts Inter — *or* self-hosted copies of those two assets for offline use.
- *Optional* (for a local web server instead of `file://`): **Python 3** or **Node.js 20+** (with `npx`).
- No package manager, virtual environment, database, or backend is required.

### 9.2 Environment Setup

No environment variables and no configuration files are used. All runtime data lives in-file as JavaScript registries (`DATA`, `FORMS`, `PRESETS`, `TEAMS`, `FOLDERS`). Get the files:

```bash
# From the repository root, on the delivered branch:
git checkout blitzy-80043409-e6c8-4df0-a740-97273bb56744
ls -1
# blitzy-integrations-page.html
# folder-sharing-prototype-v2.html
```

### 9.3 Dependency Notes

Dependencies are loaded from CDNs (no install required):

```html
<!-- Tabler Icons webfont @3.7.0 (SRI-pinned, crossorigin) -->
<link rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@3.7.0/dist/tabler-icons.min.css"
  integrity="sha384-Amzn4olt1DzKcamnc0ejzdJkVXDZuUXYZfSa4DKiYWKLMewTbAgmuDzqK0zufsXe"
  crossorigin="anonymous"/>
<!-- Google Fonts: Inter 400/500/600 -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap"
  rel="stylesheet" crossorigin="anonymous">
```

### 9.4 Application Startup

Choose any **one** option:

```bash
# Option A — open directly in a browser (simplest):
#   macOS:  open blitzy-integrations-page.html
#   Linux:  xdg-open blitzy-integrations-page.html
#   Windows: start blitzy-integrations-page.html

# Option B — Python static server (verified: HTTP 200):
python -m http.server 8000
#   then visit http://localhost:8000/blitzy-integrations-page.html
#   and       http://localhost:8000/folder-sharing-prototype-v2.html

# Option C — Node static server (verified: http-server v14.1.1):
npx --yes http-server -p 8080
#   then visit http://localhost:8080/blitzy-integrations-page.html
```

### 9.5 Verification Steps

```bash
# Syntax-check the inline JS (verified: exit 0 on both files):
node --check blitzy-integrations-page.html      # (extract inline <script> if your tooling requires)
node --check folder-sharing-prototype-v2.html

# Confirm CDN assets are reachable (verified: HTTP 200):
curl -sI "https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@3.7.0/dist/tabler-icons.min.css" | head -n 1
curl -sI "https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap" | head -n 1
```

In the browser, open DevTools → Console and confirm **zero errors**. Use the demo strip to drive **State** (Zero / Connected (mixed) / Ideal (all) / Connecting) and **Role** (Super Admin / Team Member).

### 9.6 Example Usage

- **OAuth connect (cloud):** select GitHub/GitLab/Azure DevOps → "Connect" → "Authorize on {provider}" → status transitions to **Connected** after ~1100 ms.
- **Self-hosted connect:** select GitHub Enterprise Server / GitLab Self-Managed / Bitbucket Data Center → fill URL/App ID/Secret. Try an invalid URL (`notaurl`) and an unreachable one (`https://server.invalid`) to see inline field errors; a valid reachable URL enables **Connect**.
- **Manage:** on a connected sub-card → kebab (⋯) → **Refresh connection** / **Share folder access** / **Disconnect** / **Revoke access**.
- **Folder sharing:** kebab → **Share folder access** → pick a top-level folder → add a team (inherited teams are locked with provenance; redundant grants are blocked inline) → **Done**.
- **Secondary prototype:** open `folder-sharing-prototype-v2.html` → **Add folder access** → pick folder → toggle teams → **Add** or **Save & add another** → **Done** shows the grant summary.

### 9.7 Troubleshooting

- **Icons missing / squares shown:** CDN blocked or offline. Self-host `tabler-icons.min.css` (+ its font files) and Google Fonts Inter, then repoint the `<link>` tags. All controls remain functional offline (the ADO "AZ" mark and text/CSS controls do not depend on the webfont).
- **Fonts look like the system default:** Inter CDN unreachable → falls back to `system-ui`; layout is unaffected.
- **Demo strip visible in a production build:** by design it is prototype-only — remove the `.demo` block and its CSS (marked with strip-boundary comments) for production.
- **`file://` vs server:** either works; all assets are CDN-hosted or inline, so there are no CORS issues for local files.

---

## 10. Appendices

### A. Command Reference

| Command | Purpose |
|---------|---------|
| `git checkout blitzy-80043409-e6c8-4df0-a740-97273bb56744` | Switch to the delivered branch |
| `python -m http.server 8000` | Serve the prototype locally (verified HTTP 200) |
| `npx --yes http-server -p 8080` | Alternative static server (verified v14.1.1) |
| `node --check <file>` | Validate inline JS syntax (verified exit 0) |
| `curl -sI <cdn-url> \| head -n 1` | Confirm CDN reachability (verified HTTP 200) |

### B. Port Reference

| Port | Used by | Notes |
|------|---------|-------|
| 8000 | `python -m http.server` | Suggested default for Option B |
| 8080 | `npx http-server` | Suggested default for Option C |

*(No application ports — the prototype runs entirely client-side.)*

### C. Key File Locations

| Path | Role | Lines |
|------|------|-------|
| `blitzy-integrations-page.html` | Primary prototype — full catalogue, state machine, connect dialogs, menu, folder-first sharing modal (F-001…F-012) | 479 |
| `folder-sharing-prototype-v2.html` | Secondary prototype — folder-first sharing reconciled to canonical tokens + inheritance affordances | 310 |
| `blitzy/screenshots/` | Validation evidence (13 PNGs) — intentionally not committed; out of scope | — |

### D. Technology Versions

| Component | Version | Notes |
|-----------|---------|-------|
| Tabler Icons webfont | 3.7.0 | CDN (jsDelivr), SRI-pinned in both files |
| Google Fonts — Inter | 400 / 500 / 600 | CDN |
| Node.js (validation env) | v20.20.2 | For `node --check` / `npx http-server` |
| Python (validation env) | 3.13.13 | For `python -m http.server` |
| `http-server` | 14.1.1 | Optional static server |

### E. Environment Variable Reference

**Not applicable.** The prototype reads no environment variables and uses no configuration files. All configuration is in-file via the `DATA`, `FORMS`, `PRESETS`, `TEAMS`, and `FOLDERS` JavaScript registries.

### F. Developer Tools Guide

- **Demo control strip** (top of primary page) — drive **State** (Zero / Connected (mixed) / Ideal (all) / Connecting) and **Role** (Super Admin / Team Member); a live log echoes each action. Prototype-only; remove for production.
- **Browser DevTools → Console** — validation target; should show zero errors.
- **Responsive mode** — verify at 375 / 768 / 1280 / 1440 (two-pane ≥1280; single-column reflow at 375).
- **Adding a provider** is a data change: add a `DATA` entry (+ `CATS` for a new category) — no layout code changes required.

### G. Glossary

| Term | Definition |
|------|------------|
| Company card | Visual grouping of a provider's connection variants; carries no aggregated status. |
| Sub-card (variant) | An individual connection (e.g., GitHub cloud vs GitHub Enterprise Server) with its own independent status. |
| Direct grant | A team explicitly granted access on a specific folder. |
| Inherited grant | Access a team has on a folder because of a direct grant on an ancestor folder (read-time, never stored). |
| Stable-ID grant tuple | `{connectionId, folderStableId, folderPathSnapshot, teamId}` — anchors a grant to an immutable folder identifier so rename/move is safe. |
| Redundant grant | An attempt to grant a team that already inherits access; blocked inline with its source named. |
| Disconnect | Reversible — Blitzy stops using the connection; the provider app stays installed. |
| Revoke access | Irreversible — removes Blitzy at the source; requires reinstall and re-approval. |
| Forward-reference | A downstream production target the prototype specifies but does not implement (out of scope here). |
| Push model | Administrators push access onto folders; there are no access requests or approval queues. |

---

*Completion percentage (84.1%) is computed strictly from AAP-scoped hours: 106 completed ÷ 126 total. The downstream React/TypeScript production implementation is explicitly out of scope and excluded from this figure. Brand colors applied throughout: Completed = Dark Blue `#5B39F3`, Remaining = White `#FFFFFF`.*