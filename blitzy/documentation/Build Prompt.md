# Design Solutioning Prompt — Integrations Redesign + Folder-Level Access (Blitzy)

A complete, step-by-step design brief. It tells you exactly what to design, in what order, using Blitzy's design system, with the precise tokens, layouts, states, and interactions. Follow it top to bottom; each section hands off into the next. Where a number is given (spacing, radius, hex), use it verbatim.

---

## 0. How to read this prompt

You are designing a revamp of the **Integrations settings page** plus a new **folder-level team access** capability. There are three pillars, and they nest:

1. **Structure** — reorganize the page so it scales (the container).
2. **Clubbed cards** — group connection variants under one company (what the page shows).
3. **Folder access** — let an admin grant a team a specific folder (the new capability, surfaced from a card).

Design in that order. Do not start a later pillar until the earlier one's components exist, because each builds on the last. Every component below lists: its purpose, its layout and exact tokens, its content, its states, and its interactions. Build the component first, then its states, then wire the interactions.

---

## 1. Why we are doing this (the problem)

Today the integrations page is a single flat grid of cards, one card per provider, with code tools and design tools mixed together. Each card carries one status and shares a connection as a whole. That creates six problems:

- **All-or-nothing sharing.** A team gets the entire connection or nothing. No way to scope to one folder.
- **Jumbled grid.** SCM and design tools share one undifferentiated wall that gets harder to scan as integrations grow.
- **Scattered variants.** GitHub and GitHub Enterprise sit as unrelated cards; the user can't see them as one company.
- **One status per card.** A card can't show cloud connected while self-hosted failed.
- **No clean removal.** Only Disconnect exists; the provider-side app stays installed (a security gap).
- **Doesn't scale.** Every new integration just lengthens the flat wall.

The revamp solves each of these. Keep this list in view; section 10 maps every design decision back to one of these.

---

## 2. What we are designing (scope)

**In scope**

- A categorized integrations page: a left category rail + a right canvas of company cards.
- Company cards that club connection variants as sub-cards, each sub-card independent.
- Per-integration connect dialogs (different per connection type).
- A connection 3-dot menu: Refresh connection, Share folder access, Disconnect, Revoke access.
- A folder-first sharing side modal for granting teams access to top-level folders.
- All states for every component, and the admin vs member views.

**Out of scope (do not design)**

- Multi-org or multi-account inside one connection.
- Carve-outs / exceptions under an inherited parent.
- A connect-time "share with team" prompt (sharing now lives only in the 3-dot menu).
- Free / Pro tier screens (defer; design for Team + Enterprise).

---

## 3. Design system foundations (use these exact tokens)

Pull these from the Blitzy library; do not invent new values. Every screen below references them by name.

**Type — Inter**

- Page/section title (H5): Inter SemiBold, 24px, line-height 1.3.
- Card / company name: Inter SemiBold, 18px.
- Sub-card / variant name, buttons: Inter SemiBold, 16px (button line-height 24).
- Body: Inter Regular, 16px, line-height 1.5, letter-spacing -0.3.
- Secondary / helper / descriptions: Inter Regular, 14px (Body Small), color text-secondary.
- Badge / micro labels: Inter SemiBold, 11–12px.

**Color**

- Brand / primary: `#5b39f3`. Hover: `#4f30d6`. Soft border: `#d4cbfc`. Tint fill: `#f3f0ff`.
- Text primary: `#000000`. Secondary: `#666666`. Tertiary: `#999999`.
- Border: `#d9d9d9`. Surface neutral: `#f5f5f5`. Card surface: `#ffffff`.
- Success (connected): fill `#c9fcea`, text `#005335`.
- Error (failed / destructive): fill `#ffdfdf`, text `#991010`.
- Use brand sparingly: primary buttons, active nav, selected states, links. Everything else is neutral.

**Spacing scale** — 4, 8, 12, 16, 24 (px). Use 16 as the default gap inside cards, 24 between cards, 8–12 for tight stacks.

**Radius** — 12px for cards, dialogs, inputs, buttons (rounded-md). 8px for small chips/menu rows. Pills/badges fully rounded (20px).

**Elevation** — one soft shadow for raised surfaces (cards, dialogs, menus): a low-opacity, large-blur drop shadow (Elevation Light/5). No hard borders plus heavy shadow together; pick light border + soft shadow.

---

## 4. Information architecture (the skeleton to design first)

Design the page frame before any card.

1. **Top app bar** (existing): Blitzy logo left; workspace switcher; help, settings, account right. Keep as-is.
2. **Settings tab row** (existing): Plan & Usage · Team · Integrations · Environments · Rules · Artifacts · Web search · Notifications · Profile · Password. "Integrations" is the active tab — show the active state (brand text + 2px brand underline). This row is page chrome; do not redesign it.
3. **Integrations body — two columns:**
   - **Left: category rail**, fixed width \~200–210px. A small uppercase "Categories" label (tertiary, letter-spacing), then category items. Initial items: **SCM** (active) and **Design**. Below them, a muted "More to come" label so it reads as extensible.
   - **Right: canvas**, fills remaining width, max content width \~960px. Renders the company cards for the selected category, stacked vertically with 24px gaps.

The category rail is the heart of the structure pillar. The whole point: when SCM is selected the canvas shows only code providers; when Design is selected it shows only design tools. Categories come from data, so a third category later just appears in the rail with no layout change. Design the rail and the empty canvas state ("No integrations in this category yet.") before drawing cards.

---

## 5. Component-by-component spec

### 5.1 Category rail item

- **Layout:** row, 10px gap, icon (18px) + label (14px Medium), padding 10px/12px, radius 8px.
- **States:** default (text-secondary); hover (neutral `#f5f5f5` fill); **active** (tint `#f3f0ff` fill, brand text, SemiBold). Only one active at a time.
- **Interaction:** clicking switches the canvas to that category. No page reload feel; the right side swaps.

### 5.2 Company card (the clubbing container)

This is pillar 2. A company card groups one provider's variants.

- **Surface:** white, 1px `#d9d9d9` border, 12px radius, soft shadow.
- **Header row:** padding 16px/20px, bottom hairline (`#f5f5f5`). Contains: a 34px brandmark tile (provider logo on neutral `#f5f5f5`, 8px radius), the company name (18px SemiBold), and — only if the company has more than one variant — a small caption beneath it like "2 connection types" (12px tertiary).
- **CRITICAL: no status on the company header.** Do not put an aggregate/common status here. Status belongs to each sub-card. The header is identity only.
- **Body:** the sub-cards, stacked, separated by a hairline (`#f5f5f5`). Padding 8–12px around the stack.
- **Which companies hold which sub-cards:**
  - GitHub → GitHub (cloud), GitHub Enterprise Server (self-hosted)
  - GitLab → GitLab (cloud), GitLab Self-Managed (self-hosted)
  - Azure DevOps → single sub-card
  - Bitbucket → Bitbucket Data Center (self-hosted)
  - (Design category) Figma → single sub-card, "Coming soon"
- **Principle to honor: club the look, split the access.** Variants are visually grouped under one company, but each is a separate connection with its own status, its own connect flow, and its own sharing. Never merge their status or their access.

### 5.3 Sub-card (the atomic unit — design all four states)

A sub-card is a single connection. Everything on the page resolves to it.

- **Layout:** row inside the company card. Left: 26px variant mark (logo on neutral tile). Then a text block: variant name (16px SemiBold) with the **status badge inline next to the name**, and a description line beneath (14px tertiary, e.g. "Cloud, [github.com](http://github.com)" or "Self-hosted server"). Right, pushed to the edge: the action area.
- **The four states (design each):**
  1. **Not connected** — no badge; action area = one primary **Connect** button.
  2. **Connecting** — badge "Connecting" (brand tint fill, brand text, 15px spinner); no buttons.
  3. **Connected** — badge "Connected" (success fill/text); action area = **Manage** (outline button) + a **3-dot** icon button.
  4. **Failed / expired** — badge "Connection failed" (error fill/text); action area = **Reconnect** (primary button) + 3-dot.
- **Sibling independence:** in one company card, design the case where GitHub is Connected and GitHub Enterprise Server is Failed at the same time. This is the proof that status is per sub-card.

### 5.4 Status badges

- Pill, fully rounded, 11–12px SemiBold, icon + label, \~3px/9px padding.
- Connected: `#c9fcea` / `#005335`, check icon. Failed: `#ffdfdf` / `#991010`, alert icon. Connecting: `#f3f0ff` / `#5b39f3`, spinner. Coming soon: `#f5f5f5` / `#999999`, no icon.

### 5.5 Buttons

- Primary: brand `#5b39f3` fill, white text, 16px SemiBold, 12px radius, \~9px/18px padding. Hover `#4f30d6`.
- Outline (Manage): white fill, brand text, soft brand border `#d4cbfc`. Hover tint `#f3f0ff`.
- Ghost (Cancel): transparent, secondary text, neutral border.
- Destructive (in confirmations): error `#991010` fill, white text.
- 3-dot: 34px square, white, neutral border, secondary icon; hover neutral fill.
- Disabled: \~45% opacity, not-allowed.

### 5.6 Connect dialog — CLOUD variant (GitHub, GitLab, Azure DevOps)

This is the simplest connect path. Design a centered modal over a 40% scrim.

- **Header:** provider logo (22px) + title "Connect {Provider}".
- **Body:** one short paragraph (14px secondary): "You will be redirected to {Provider} to authorize Blitzy. After you approve, you will return here and the connection will be active."
- **Footer (right-aligned):** Cancel (ghost) + "Authorize on {Provider}" (primary, with an external-link icon).
- **Interaction:** Authorize → modal closes → sub-card goes Connecting → Connected. No further fields.

### 5.7 Connect dialog — SELF-HOSTED / DATA CENTER variant (GitHub Enterprise Server, GitLab Self-Managed, Bitbucket Data Center)

This is the rich one. Design it carefully; it differs per provider only in field labels and helper text.

- **Header:** provider logo + title "{Provider} connection" (e.g. "Bitbucket Data Center connection").
- **Top helper box:** a tint `#f3f0ff` block, 12px radius, 13px text: "First, create an application in {Provider}" with a brand "Learn how ↗" link. This sets up the OAuth-app prerequisite.
- **Fields (stacked, each: bold 13px label, input, 12px tertiary helper beneath):**
  - Bitbucket Data Center: `Bitbucket URL` (placeholder `https://bitbucket.yourcompany.com`, helper "Your organization's Bitbucket instance") · `Application ID` (helper "Administration → Application Links") · `Secret` (helper "Shown once when creating the application").
  - GitHub Enterprise Server: `Server URL` (`https://github.yourcompany.com`, helper "Your organization's GitHub Enterprise Server") · `Client ID` (helper "Developer settings → OAuth Apps") · `Client secret` (helper "Shown once when registering the app").
  - GitLab Self-Managed: `GitLab URL` (`https://gitlab.yourcompany.com`, helper "Your organization's GitLab instance") · `Application ID` (helper "Preferences → Applications") · `Secret` (helper "Shown once when creating the application").
- **Input style:** 40px tall, 1px `#d9d9d9` border, 12px radius; focus = brand border + 3px brand-tint focus ring.
- **Footer:** Cancel (ghost) + Connect (primary). **Connect is disabled until all required fields are filled** — design the disabled state, it's the greyed button.
- **Interaction:** Connect → modal closes → Connecting → Connected.

### 5.8 The 3-dot menu (on a connected sub-card)

- **Surface:** white popover, 1px border, 12px radius, soft shadow, \~210px wide, anchored to the 3-dot, right-aligned.
- **Rows (13.5px, icon + label, 9px/10px padding, 6px radius, hover neutral):**
  1. Refresh connection (refresh icon)
  2. Share folder access (people icon)
  3. — hairline divider —
  4. Disconnect (unlink icon, error text)
  5. Revoke access (shield-x icon, error text)
- The two destructive rows sit below the divider so they read as a separate, heavier group.

### 5.9 Disconnect confirmation

- Modal. Title "Disconnect {variant}?". Body (14px secondary): "Blitzy will stop using this connection. Folder access granted to teams from it, and any project using it, will break. The app stays installed on the server, so you can reconnect later without re-approving." Footer: Cancel (ghost) + Disconnect (destructive).
- Tone: serious but reversible.

### 5.10 Revoke access confirmation (this is the renamed "uninstall")

- Modal. Title "Revoke access to {variant}?". Body: "This removes Blitzy from the server entirely and revokes its permissions at the source. All folder grants and connected projects break. To use it again you must reinstall and re-approve from scratch." Footer: Cancel + Revoke access (destructive).
- Tone: heavier than Disconnect. Make the consequence unmistakable. The naming matters: "Disconnect" = pause, app stays; "Revoke access" = full removal at the source. Never label this "Uninstall" in the UI.

### 5.11 Folder sharing side modal (pillar 3 — the core new capability)

Opened from the 3-dot → Share folder access. **Folder-first**: pick a folder, then assign teams.

- **Surface:** a wide modal (or right-side panel). Header: provider logo + "{variant} — folder access". A one-line lede (14px secondary): "Grant a team access to a top-level folder. Everything inside inherits it. This connection has its own access, separate from other connections."
- **Two panes inside a bordered, 12px-radius container:**
  - **Left pane (≈42%, neutral** `#f5f5f5`**):** the folder list. **Only top-level folders are selectable** (the level above repo: GitHub org, ADO project, GitLab group/subgroup, Bitbucket project). Each folder row: folder icon + name + a small count of teams with access. Selected row = white fill + brand text. (If you show repos/branches for context, render them muted and non-selectable; selecting one shows a note "Access is granted at the folder level.")
  - **Right pane (≈58%, white):** for the selected folder —
    
    - Section "Teams with access to {folder}": list each team with a people icon. **Directly granted** teams have a brand "Remove" link. **Inherited** teams (granted on a parent) are shown muted, with a lock icon and "from {parent}", and have **no Remove** here.
    - Section "Add a team": a **search input** (not a dropdown — orgs have many teams), then matching team rows, each with a "+" to add as a direct grant.
  - **Footer:** Done (primary).
- **The rules to encode visually:**
  - **Push, no requests:** there is no "request" or "pending" anywhere; the admin just adds.
  - **Inheritance flows down:** adding a team to a parent shows it as inherited on every child.
  - **No carve-outs:** there is no way to remove a team from a child when it's inherited — the only control is on the parent.
  - **Redundant grant blocked:** if the admin tries to add a team that already inherits the folder, block it and show an inline note: "{team} already has access from {parent}. Remove it there to change access." Design this blocked/inline-note state.
  - **Per-connection:** this modal is scoped to one sub-card. GitHub and GitHub Enterprise Server each open their own, with their own teams. Make the header make that obvious.

---

## 6. Page-level states (compose from the sub-cards)

Design the canvas in these four states so the team sees the full range:

- **Zero state:** category selected, every sub-card "Not connected" (all show Connect). Optional light hint that nothing is connected yet.
- **Connecting:** at least one sub-card mid-connect (spinner badge).
- **Connected (mixed):** the realistic working view — some connected, at least one failed, some not connected. This is the primary state to polish.
- **Ideal:** every sub-card in the category connected and healthy.

---

## 7. Roles (design two views)

- **Super Admin:** full controls — Connect, Manage, the 3-dot (Refresh, Share folder access, Disconnect, Revoke access).
- **Team Member:** read-only. Sub-cards show **status only**; no Connect, Manage, 3-dot, or sharing. Design this stripped view explicitly so it's clear members can see what's connected but not change it.
- Tiers: design for Enterprise and Team. (Free/Pro deferred.)

---

## 8. Flows (the click paths to prototype)

1. **Connect (cloud):** SCM → company card → Connect on sub-card → authorize modal → Authorize → Connecting → Connected (Manage + 3-dot appear).
2. **Connect (self-hosted):** Connect → credentials modal → fill URL + app id + secret → Connect (enables) → Connecting → Connected.
3. **Grant folder access:** connected sub-card → 3-dot → Share folder access → side modal → select top-level folder → search team → add → team appears as direct; children show it inherited.
4. **Revoke vs disconnect:** 3-dot → Disconnect (confirm, app stays) OR Revoke access (heavier confirm, removed at source).
5. **Member view:** same page, controls hidden, status visible.

---

## 9. Edge cases to design

- Mixed status in one company (connected + failed siblings).
- Self-hosted connect with an invalid/unreachable URL → inline field error, stay on form.
- OAuth cancelled → sub-card returns to Not connected with retry.
- Adding a team that already inherits → blocked inline note.
- A granted folder deleted in the SCM → the grant shows a broken/error state (the only error case; rename and move stay intact silently).
- Disconnect/Revoke on a connection that has folder grants or dependent projects → the confirmation warns those break.
- Empty category → friendly empty state, not a broken page.
- Coming-soon integration (Figma) → non-actionable card with a "Coming soon" badge.

---

## 10. Pain point → design decision (keep the team oriented)

- Jumbled grid → **category rail** (one type at a time).
- Scattered variants → **company cards with sub-cards**.
- One common status → **status on each sub-card**.
- All-or-nothing sharing → **folder-first sharing modal**.
- No clean removal → **Revoke access** beside Disconnect.
- Doesn't scale → **data-driven categories**.
- Can't see who's shared (ARUI-3268 / ARUI-3232) → **teams listed per folder, direct vs inherited**.

---

## 11. What to deliver

- The page frame (tabs + category rail + canvas) in all four page states.
- Company card and sub-card components with all four sub-card states and the mixed-sibling case.
- Both connect dialogs (cloud + self-hosted), including the disabled-Connect state.
- The 3-dot menu, plus Disconnect and Revoke access confirmations.
- The folder-first sharing side modal, including the inherited and blocked-redundant states.
- The Team Member read-only view.
- A short tokens page proving every value above maps to the Blitzy library.

Use Blitzy's existing components and tokens throughout. When something isn't in the library yet (e.g. the company card or the sharing side modal), build it from existing primitives — surfaces, the badge, the button set, inputs, the menu — so it reads as native Blitzy, not a new visual language.