# Blitzy Feature Prompt — Integrations Redesign & Folder-Level Team Access

> Paste into Blitzy as a New Feature prompt. Structured WHY / WHAT / HOW per the Blitzy template, with explicit IN SCOPE / OUT OF SCOPE and named integration points. Replace any value in {curly braces} with the exact identifier from the repo before running.

---

## 1. Vision & Purpose (WHY)

Today an SCM connection in Blitzy can only be shared with a team as a whole. It is all-or-nothing: a team either sees the entire connection or none of it. Enterprises need a team to see only the folders that belong to its work.

This feature does two things:

1. **Restructures the Integrations settings page** so it scales as providers grow, and so each connection variant (cloud vs self-hosted) has its own independent status and lifecycle.
2. **Adds folder-level access**: a super admin grants a team access to a specific top-level folder inside a connected SCM, instead of the whole connection.

Outcome: scoped, scalable, self-service integration management, and granular team access that the current whole-integration sharing cannot express.

---

## 2. Core Requirements (WHAT)

### IN SCOPE

**A. Integrations page restructure**

- A category navigation (left rail) that renders categories from data, not hard-coded. Initial categories: `SCM` and `Design`. The page must support adding categories later without layout changes.
- Within a category, render one **company card** per provider. A company card contains one or more **sub-cards**, one per connection variant:
  
  - GitHub → `GitHub` (cloud), `GitHub Enterprise Server` (self-hosted)
  - GitLab → `GitLab` (cloud), `GitLab Self-Managed` (self-hosted)
  - Azure DevOps → single sub-card
  - Bitbucket → `Bitbucket Data Center` (self-hosted)
  - Design category → `Figma` (rendered as "Coming soon" until available)
- **Status lives on the sub-card, not the company card.** Each sub-card shows its own status independently (e.g. GitHub connected while GitHub Enterprise Server failed). Do NOT show a single aggregated status for the company.

**B. Per-integration connect flow**

- Clicking Connect on a sub-card opens a connection dialog whose contents depend on the connection type:
  
  - **Cloud (GitHub, GitLab, Azure DevOps):** an authorize step that redirects to the provider's OAuth, then returns and marks the sub-card connected.
  - **Self-hosted / Data Center (GitHub Enterprise Server, GitLab Self-Managed, Bitbucket Data Center):** a credentials form collecting the instance URL plus the provider's app credentials, with a "first, create an application in {provider} / Learn how" helper link. Example fields for Bitbucket Data Center: `Bitbucket URL`, `Application ID`, `Secret`. The Connect button stays disabled until required fields are filled.
- On success the sub-card shows status `Connected`, a `Manage` action, and a 3-dot menu. There is **no** connect-time "share with team" prompt (sharing is handled from the 3-dot menu).

**C. Connection 3-dot menu (on a connected sub-card)**

- `Refresh connection` — re-validates / refreshes the existing connection.
- `Share folder access` — opens the folder-level sharing flow (section D).
- `Disconnect` — Blitzy stops using the connection; the provider-side app/install remains. Reversible without re-approval. Requires a confirmation that warns folder grants and dependent projects will break.
- `Revoke access` — the "uninstall" action, renamed. Removes Blitzy from the provider entirely and revokes permissions at the source; requires reinstall + re-approval to return. Requires a heavier confirmation.

**D. Folder-level team access (the core capability)**

- A super admin assigns a team access to a **top-level folder** inside a connection. Repos and branches are NOT grantable; only folders (the level above repo: GitHub org, ADO project, GitLab group/subgroup, Bitbucket project/workspace).
- **Push model:** the admin assigns directly. No access request, no approval queue.
- **Inheritance:** granting a parent folder grants everything nested under it automatically.
- **No carve-outs:** access under a parent is all-or-nothing; an admin cannot grant a parent and then hide a child.
- **Source and destination:** a granted folder is usable by the team both as a project source repo and as the destination for generated code.
- **Multiple grants:** a team can hold multiple folders, across multiple connections. Each connection's grants are independent.
- **Visibility of inherited access:** when viewing a child folder, teams that have access via a parent are shown as inherited with the source parent named, and are not removable on the child (the grant must be changed on the parent).
- **Redundant grant blocked:** attempting to grant a team a folder it already inherits is prevented with an inline message pointing to where the access actually lives.
- **Grant storage by stable ID:** persist each grant against the provider's permanent folder/node ID, not its name or path (see HOW → Data model).
- **Sharing UI:** a folder-first panel — folder tree/list on the left, the selected folder's teams (direct + inherited) and a team search-to-add on the right.

### OUT OF SCOPE (do not build)

- Multi-org or multi-account per connection (one account per sub-card).
- Cross-org folder grants.
- Carve-out / exception rules under an inherited parent.
- The legacy connect-time "Just me / Share with team" whole-integration prompt (replaced).
- Free / Pro tier behavior for this feature (deferred; gate to Team + Enterprise).
- Backend changes to how the SCM tree is fetched (reuse the existing tree).

---

## 3. Technical Implementation (HOW)

> Confirm these identifiers against the repo. Where the current code differs (e.g. GitHub Enterprise Server or Bitbucket Data Center are not yet represented as connection types), follow the existing provider/adapter pattern to add them rather than inventing a new one.

### Integration points (name the exact files/contracts)

- **Integrations page:** `src/panel/workspace/settings/integrations.tsx` (route `/workspace/settings/integrations`). Restructure into category nav + company cards + sub-cards here. Keep it available to authenticated users; gate management actions by role (see Business Requirements).
- **Provider types /** `SvcType`**:** existing values `GITHUB`, `AZURE_DEVOPS`, `GITLAB`, `GITLAB_SELF_HOSTED`. Extend the provider model to represent the self-hosted GitHub (GitHub Enterprise Server) and Bitbucket Data Center variants, following the same adapter pattern used for `GITLAB_SELF_HOSTED`.
- **Adapters:** the existing GitHub / Azure DevOps / GitLab adapters that normalize provider data to the shared `TreeNode` contract. Reuse the normalized tree for the folder picker. Respect `MAX_GITLAB_DEPTH = 20`.
- **Existing sharing contract:** `IntegrationTeamShareRequest` and `bulkUpdateIntegrationTeamAccess` currently share a whole integration with a team. Extend (or add alongside) a contract that references a **folder/node ID** so a grant targets a folder, not the whole integration. Migration of existing whole-integration shares must be defined (see Open Questions).
- **Connection lifecycle:** `Disconnect` exists today. Add `Revoke access` (uninstall) as a distinct action; they are different operations. (Tickets: ADO uninstall `ABK-939`; silent token expiry `ABK-2730`.)

### Data model (folder grant)

- A grant = `{ connectionId, folderStableId, folderPathSnapshot, teamId }`.
- `folderStableId` is the provider's permanent internal ID (never changes on rename/move). `folderPathSnapshot` is for display only and is refreshed from the live tree.
- Behavior on tree changes, resolved by reading the live tree by `folderStableId`:
  
  - **Rename:** no break. Same ID; update displayed path; no error.
  - **Move:** no break. Same ID; update displayed path; no error.
  - **Delete:** the ID resolves to nothing → mark the grant as broken and surface an error state. This is the only error case.
- Inheritance is computed: a team has access to a folder if it has a direct grant on that folder OR a grant on any ancestor folder.

### Frontend / state

- Reuse the existing design system tokens (Inter; brand `#5b39f3`; radius 12; spacing 4/8/12/16/24; success `#c9fcea`/`#005335`; error `#ffdfdf`/`#991010`; borders `#d9d9d9`). Match existing card, badge, button, and dialog patterns.
- Sub-card states: `not connected`, `connecting`, `connected`, `failed/expired`. One primary action per state (Connect / spinner / Manage + 3-dot / Reconnect + 3-dot).
- Connect dialog is config-driven per provider variant (oauth vs credentials form with field list).

---

## 4. User Experience & Flows

**Connect (super admin)**

1. Integrations → pick category in left nav → find company card → click Connect on the target sub-card.
2. Cloud: authorize on provider; Self-hosted: fill instance URL + app credentials, submit.
3. Sub-card shows `Connecting`, then `Connected` with Manage + 3-dot. No share prompt at connect.

**Grant folder access (super admin)**

1. Connected sub-card → 3-dot → Share folder access.
2. Folder-first panel opens. Select a top-level folder.
3. Search a team, add it → direct grant on that folder; everything inside inherits.
4. On a child folder, inherited teams show with their source parent and are read-only there. Re-granting an inherited team is blocked.

**Consume (team member)**

- In a project, the source and destination pickers show only folders the member's team has been granted (across connections). No access request path.
- Management actions (Connect, Manage, Disconnect, Revoke access, Share folder access) are hidden for team members; they see status only.

---

## 5. Business Requirements (access control, rules, priority)

- **Roles:** Super Admin grants/revokes folder access and manages connections. Team Member is read-only on this page and consumes granted folders. (Team-tier admin grant capability: see Open Questions.)
- **Tiers:** Enterprise and Team. Note: team management is currently Enterprise-gated; confirm gating for the grant action on Team tier before enforcing.
- **Rules:** push-only (no requests); top-level folders only; inheritance flows down; no carve-outs; one account per sub-card.
- **Priority:** (1) page restructure + per-sub status, (2) folder-level grant model + sharing UI, (3) Revoke access action + confirmations.

---

## 6. Edge Cases

- Mixed status within a company (one sub-card connected, sibling failed) renders correctly per sub-card.
- Self-hosted connect with an unreachable/invalid instance URL → form-level error; stay on the form.
- OAuth cancelled/denied → sub-card returns to `not connected` with retry.
- Granting a team a folder it already inherits → blocked with explanatory inline message.
- Granted folder deleted in the SCM → grant shows broken/error state (only error case).
- Disconnect or Revoke on a connection with active folder grants or dependent projects → confirmation warns those break.

---

## 7. Testing Requirements

- **Unit:** inheritance resolution (direct vs inherited), redundant-grant blocking, grant resolution by stable ID across rename/move/delete.
- **Integration:** connect flows per provider variant (oauth and credentials form), grant/revoke writing through the folder-aware sharing contract, Disconnect vs Revoke access distinct effects.
- **E2E:** super admin connects → grants a folder → team member sees only granted folders in source/destination pickers; member cannot see management actions.

---

## 8. Acceptance Criteria

- Categories render from data; adding one requires no layout change.
- Each sub-card shows its own independent status.
- Connect dialog content differs by provider variant; self-hosted requires instance URL + credentials.
- 3-dot menu shows Refresh connection, Share folder access, Disconnect, Revoke access, with correct confirmations and distinct Disconnect vs Revoke behavior.
- A team granted a parent folder can use any folder inside it as source and destination; cannot see ungranted folders.
- Rename and move of a granted folder do not break access; delete surfaces an error.

---

## 9. Open Questions for Engineering (resolve before/while building)

1. Does the current code persist a share by stable folder ID or by name/path? This decides whether rename/move are safe as specified.
2. How is GitHub Enterprise Server represented today (separate connection type, host variant of `GITHUB`, or not yet built)? Same question for Bitbucket Data Center.
3. Migration: what happens to existing whole-integration `IntegrationTeamShareRequest` shares when folder-level grants ship — auto-map to the connection root, or grandfather?
4. On a deleted granted folder, who is notified (granting admin, consuming member, or both), and is a dependent broken project in scope here or a separate fix?
5. Team-tier gating for the grant action, given team management is Enterprise-gated today.