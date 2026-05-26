---
name: pj-confluence-master-sync
description: "Use when a PJ wants Codex to run or inspect a Confluence-only project workspace sync: understand a PJ-designed Baseline, discover Master File, Draft Master Copy, Action Log, and module pages, check module updates, read read-only module pages, summarize changes and conflicts, merge module content into Draft Master Copy only, query prior Action Log runs, and log actions without requiring a server or database. Also use for scheduled or manual Codex runs that maintain Confluence masterfile/module collaboration workflows."
---

# PJ Confluence Master Sync

Operate a Confluence-only project workspace where Confluence pages are the shared state layer. The skill must support different PJ-designed Baselines; do not assume a universal project schema.

## Core Rules

- Treat module pages as read-only sources.
- Treat Draft Master Copy as a merge proposal, not synced truth.
- Write automated merge output only to Draft Master Copy.
- Do not write Live Master unless the PJ explicitly asks after reviewing the Draft in the same task.
- Never silently erase Live Master content. Schema-driven removals must be shown as proposed removals in Draft and require explicit PJ approval at publish time.
- Do not silently guess uncertain Baseline mapping, target placement, or conflicts.
- Do not store, print, or hard-code credentials, local paths, cookies, tokens, or machine-specific setup.
- Prefer Confluence page IDs, titles, links, version numbers, timestamps, and page body hashes as state; do not require a server or database.
- Track Draft proposal state separately from published Live Master state.
- Record material actions in Action Log when available; if logging fails, return the log row for the PJ.

## Workspace Shape

Expect one Project Folder page containing or linking:

- `Master File`: live/reference master content.
- `Baseline`: PJ-owned config and sync state.
- `Draft Master Copy`: only automated write target.
- `Action Log`: action records.
- Module pages: usually child pages under Master File, unless Baseline defines another source.

If a project is missing a core page, stop or ask before creating it. If creating pages is requested, create them as Project Folder children and keep module pages under Master File unless the Baseline says otherwise.

## Run Modes

### Access Readiness Gate

Before reading or writing project pages, classify the current access mode:

- `manual_browser`: authenticated browser/Chrome session is available.
- `rest_token`: approved non-interactive Confluence REST credential is available.
- `connector`: approved Confluence connector/plugin is available.
- `blocked`: no usable auth path is available.

Manual merge can run with `manual_browser`, `rest_token`, or `connector`. Scheduled runs require `rest_token` or `connector`; if only `manual_browser` is available, return `blocked` before touching pages.

### New Project Onboarding

1. Check Confluence access and whether the current run has a usable auth method.
2. Read Project Folder and Baseline.
3. Return Baseline Understanding.
4. Ask PJ to confirm ambiguous pages, modules, elements, or placements.
5. Do not merge until the Baseline Understanding is confirmed or intentionally accepted by PJ.

### Existing Project Manual Merge

1. Read latest Live Master, Baseline, Draft Master Copy, Action Log, Draft proposal state, and published state.
2. Record Live Master base version/hash and Baseline schema version/hash for this run.
3. Compare current Baseline schema against the last Draft proposal and last published state.
4. If schema changed, revalidate all included modules and all previously published managed elements; do not rely only on changed module pages.
5. If schema did not change, detect changed/new/unreadable modules and revalidate changed modules plus any dependent placements.
6. Build a Draft proposal from current Live Master plus schema-eligible module updates.
7. Classify every affected target as `new`, `updated`, `removed_proposed`, `unchanged`, `excluded_by_schema`, `conflict`, or `human_preserved`.
8. Write the proposal to Draft Master Copy only when mapping is safe or PJ explicitly accepts listed ambiguity.
9. Update Action Log and Draft proposal state in Baseline when available. Do not update published state until Live Master is actually published.

### Scheduled Codex Run

Use the same flow as manual merge, but first verify that scheduled Codex has `rest_token` or `connector` access. Browser session-only access may pass manual tests but fail scheduled tasks. If scheduled auth is unavailable, return `blocked` with the missing credential/plugin/instruction placeholder.

### Action Log Query

Use this mode when the PJ asks what happened before, whether a merge ran, what changed last time, who triggered an action, or why a run was blocked.

1. Read the Project Folder or Baseline to find Action Log.
2. Read Action Log rows.
3. Filter by the user's question: latest run, date range, module, action_type, result, run_id, target_page, or blocked/needs_review status.
4. Return a concise answer with links to affected pages and the next PJ action.
5. Do not modify project pages during log query mode.

## Baseline Understanding

Normalize any PJ Baseline into five areas:

1. `project_pages`: Master File, Baseline, Draft Master Copy, Action Log.
2. `modules`: module name, source page link, include flag, order, owner/status if available.
3. `content_elements`: fields, layers, sections, workflow items, or content blocks each module should provide.
4. `master_placement`: where each module/element lands in Draft/Master.
5. `sync_state`: page version, updated time, content hash, last synced time, or blank for first run.

When possible, also extract a simple `master_structure` map:

- page/project layer
- module layer
- schema element layer
- content block layer
- module-end out-of-schema placeholder layer

Return this contract before merge:

```text
Baseline Understanding
- Status: confirmed / needs_review / blocked
- Project pages found
- Modules found
- Content elements found
- Master/Draft structure found
- Master placement found
- Sync state found
- Ambiguities
- Missing required items
- Recommended next action
```

Use `references/baseline-mapping.md` when interpreting flexible Baseline designs.

## Update Detection

For each included module, compare current page state against Baseline sync state or the latest Action Log entry:

- page ID
- page title
- Confluence version number
- last updated time/editor if available
- hash of normalized storage/body content when available

If no prior state exists, treat the module as `first_run_collect`.

## Merge Planning

Before writing Draft Master Copy, return:

- which module pages will be collected
- which Live Master version/hash the proposal is based on
- which Baseline schema version/hash the proposal uses
- whether schema changed since the last Draft proposal or last published state
- what changed in each module
- what will be placed where in Draft Master Copy
- what would be added, updated, removed, or left unchanged in Live Master if the Draft is published
- what Live Master content is preserved as human-added or unmapped
- missing required elements
- conflicts or low-confidence mappings
- whether writeback is safe

Conflict examples:

- two modules map to the same target element with incompatible content
- required element is missing from a module page
- module page changed but its Baseline target is unclear
- Baseline and page title/module name disagree
- target section cannot be found in Draft/Master and no fallback is defined

## Draft Master Writeback

Write only after merge planning is safe or PJ explicitly says to proceed with listed ambiguity.

When writing:

- Read latest Draft Master Copy immediately before update.
- Read latest Live Master and Baseline immediately before building the Draft proposal.
- Preserve content outside managed draft output.
- Use one managed section marked by visible Confluence-safe labels `PJ_CONFLUENCE_MASTER_SYNC_START` and `PJ_CONFLUENCE_MASTER_SYNC_END`, for example bold paragraph markers in storage HTML. Do not rely on HTML comments, because Confluence may strip them.
- If the managed section exists, replace only that section.
- If no managed section exists, append one managed section instead of overwriting existing PJ text.
- Include proposal metadata, version guardrails, inherited Master body with inline proposal markings, module-end out-of-schema content, change ledger index, conflicts, and publish instructions.
- Make Proposed Master Content the primary review surface. It must first inherit the full current Live Master structure/content, then insert proposed changes at the correct Baseline target placement.
- Preserve and expose the content layers used by the Master. At minimum, Draft should make module sections, schema element sections, content blocks, and module-end out-of-schema placeholders visible enough for PJ to compare against Baseline schema.
- Do not put a standalone Draft scaffold, placeholder module list, or "Draft Output Area" before the inherited Master-shaped proposal. If an old Draft scaffold exists, remove or replace it during Draft generation unless the PJ explicitly asks to preserve it.
- Keep run metadata, guardrails, and Change Ledger as compact supporting sections after the inherited Master-shaped proposal, or in a clearly separated metadata area. They must not replace or visually dominate the Master-shaped review body.
- Highlight changed proposal content with stable text labels: `[NEW]`, `[UPDATED]`, `[REMOVED - PROPOSED]`, `[EXCLUDED BY SCHEMA]`, `[CONFLICT]`, and `[HUMAN PRESERVED]`. Use Confluence panels/status macros only when available, but keep the text labels even if macros are used.
- Render additions inline at the target placement with `[NEW]`.
- Render updates inline at the target placement with `[UPDATED]`, showing the previous Master content and proposed replacement near each other.
- Render schema-driven deletions inline at the original Master location with strikethrough, for example `<s>[REMOVED - PROPOSED] old content</s>`.
- Put schema-ineligible or unmapped module content at the end of the related module section as `[EXCLUDED BY SCHEMA] Module End Notes`, not only in a detached table. Do not treat it as publishable managed Master content unless the PJ updates Baseline schema or explicitly approves inclusion with a clear target.
- Keep Change Ledger as an index/summary after the inherited Proposed Master body. The ledger should help PJ navigate changes, not replace the inline Draft review.
- Show schema-driven removals as proposed removals only. Do not delete them from Live Master during Draft generation.
- Leave Live Master unchanged.

Use `references/proposal-versioning.md` for Draft proposal structure, state rules, schema-change handling, and publish guardrails.

## Live Master Publish

Only publish after PJ explicitly approves the Draft proposal.

Before publishing:

- Re-read Draft Master Copy, Live Master, Baseline, and Action Log.
- Verify the Draft proposal references the current Live Master version/hash. If Live Master changed after Draft generation, block publish and regenerate/rebase the Draft proposal.
- Verify the Draft proposal references the current Baseline schema version/hash. If Baseline schema changed after Draft generation, block publish and regenerate the Draft proposal.
- Copy only the approved Proposed Master applicable content into Live Master.
- Preserve Live Master content outside managed sync sections.
- Do not remove human-added notes or unmanaged content unless the PJ explicitly approved the specific removal in the same publish request.
- Treat `removed_proposed` items as removals requiring explicit PJ approval. If approval is missing, keep them in Live Master and list them in the publish result as not removed.
- Exclude `[EXCLUDED BY SCHEMA] Module End Notes` from Live Master publish unless the PJ explicitly approves them and the Baseline target/schema is clear.
- After successful publish, update published state in Baseline and append Action Log. Keep Draft proposal state for traceability.

## Action Log

Append or update Action Log with these fields:

- `run_id`
- `log_time`
- `actor`
- `mode`
- `action_type`
- `project`
- `target_page`
- `source_page_ids`
- `source_versions`
- `draft_before_version`
- `draft_after_version`
- `live_master_base_version`
- `baseline_schema_version`
- `draft_run_id`
- `publish_run_id`
- `summary`
- `result`
- `status_reason`
- `next_action`

Common `action_type` values:

- `readiness_check`
- `config_understanding`
- `update_check`
- `module_collect`
- `draft_merge`
- `draft_proposal`
- `live_publish`
- `sync_state_update`
- `blocked`

Use `references/output-templates.md` for response and log templates.
Use `references/log-query.md` when the PJ asks questions about prior runs or Action Log records.

## Tool And Credential Notes

Use whatever Confluence access method is available in the current Codex environment:

- authenticated browser/Chrome session for manual tests
- Confluence REST API when an approved token/session is available
- user-provided plugin/connector if installed later

For production or scheduled usage, leave placeholders in instructions for:

- Confluence base URL
- target space key
- auth method
- credential source name
- whether the run is manual or scheduled

Never include actual credential values in the skill, responses, logs, or Sheet tabs.

## User-Facing Result

For every run, answer concisely:

```text
Confluence Master Sync Result
- Status:
- Project:
- Pages used:
- Modules checked:
- Modules changed:
- Draft Master Copy:
- Conflicts:
- Missing / low confidence:
- Action Log:
- Next PJ action:
```

## References

- Read `references/baseline-mapping.md` for Baseline normalization and mapping rules.
- Read `references/confluence-workflow.md` for workspace/state/writeback rules.
- Read `references/proposal-versioning.md` for Draft proposal, schema-change, and publish rules.
- Read `references/log-query.md` for Action Log query examples and filters.
- Read `references/output-templates.md` for result and Action Log formats.
