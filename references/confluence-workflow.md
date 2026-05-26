# Confluence Workflow Reference

Use this for page structure, state storage, and writeback behavior.

## Recommended Page Structure

```text
NPH
└── Project Folder
    ├── Master File
    │   ├── Module Page 1
    │   ├── Module Page 2
    │   └── Module Page N
    ├── Baseline
    ├── Draft Master Copy
    └── Action Log
```

Module pages are sources. Draft Master Copy is the automated output. Master File is live/reference content.

## No Database State

Store state in Confluence:

- Baseline: mapping, module list, source page links, target placement, latest sync state.
- Action Log: each run, result, and history.
- Draft Master Copy: latest generated merge proposal, unresolved conflicts, held-out content, and proposed removals.

If sync state is missing, use a first-run full collection and then write the discovered state back to Baseline or Action Log when allowed.

## Readiness Check

Before a new project or scheduled run, confirm:

- Confluence base URL and space are reachable.
- Project Folder can be read.
- Baseline can be read.
- Module pages can be read.
- Draft Master Copy can be written.
- Action Log can be written or a log row can be returned.
- Scheduled runs have non-interactive auth, not only a live browser session.

Classify auth before proceeding:

| access_mode | Manual run | Scheduled run | Notes |
|---|---:|---:|---|
| manual_browser | OK | Block | Uses current authenticated browser session. |
| rest_token | OK | OK | Preferred for scheduled Codex. |
| connector | OK | OK | Preferred if a Confluence connector exists. |
| blocked | Block | Block | Ask PJ/Admin for auth path. |

## Update Detection

For each module page, collect:

- page ID
- title
- URL
- version number
- last modified time/editor if available
- normalized content hash if available

Compare against Baseline sync state. Mark each module:

- `unchanged`
- `changed`
- `new`
- `first_run_collect`
- `unreadable`
- `missing`

Also collect current Live Master version/hash and current Baseline schema version/hash before deciding scope.

If Baseline schema changed since the latest Draft proposal or latest published state, revalidate all included modules and all previously published managed elements. Schema change can make content newly eligible or newly ineligible, so module page version alone is not enough.

## Draft Writeback Pattern

Use one safe managed section with visible Confluence-safe labels. Do not use HTML comments for the markers, because Confluence may strip comments from storage. The Draft is a merge proposal generated from current Live Master plus eligible module updates:

```html
<p><strong>PJ_CONFLUENCE_MASTER_SYNC_START</strong></p>
Draft Merge Proposal
Run ID:
Run time:
Live Master base version/hash:
Baseline schema version/hash:
Modules collected:
Schema changed since last proposal/publish:

Proposal Summary
Version Guardrails
Proposed Master Content: full current Master body plus inline labels at target placements
Module End Notes For Out-of-Schema Content
Change Ledger: navigation/index summary only
Conflicts And PJ Decisions Needed
Publish Command Guidance
<p><strong>PJ_CONFLUENCE_MASTER_SYNC_END</strong></p>
```

If both markers exist, replace only the content between them. If no markers exist, append one managed section. If only one marker exists, stop with `blocked` because the Draft managed section is corrupted. Preserve user content outside the managed output.

The Proposed Master Content must inherit the complete current Live Master body first. Then add inline changes at the correct Baseline target locations. The Change Ledger must classify affected targets as `new`, `updated`, `removed_proposed`, `unchanged`, `excluded_by_schema`, `conflict`, or `human_preserved`, but it is an index only. Use visible labels `[NEW]`, `[UPDATED]`, `[REMOVED - PROPOSED]`, `[EXCLUDED BY SCHEMA]`, `[CONFLICT]`, and `[HUMAN PRESERVED]` in the Draft body so PJ can scan what changed in context.

If content is deleted or becomes schema-ineligible, keep it visible at the original Master location with strikethrough, for example `<s>[REMOVED - PROPOSED] old content</s>`. If content is outside schema, add it at the end of the related module section as `[EXCLUDED BY SCHEMA] Module End Notes`, not only in a detached held-out table.

## Baseline Sync State Pattern

Keep separate latest-state tables in Baseline.

Latest Draft Proposal State:

| draft_run_id | draft_page_id | draft_version | live_master_base_version | live_master_base_hash | baseline_schema_version | baseline_schema_hash | module_versions | generated_at | result |
|---|---|---:|---:|---|---:|---|---|---|---|

Latest Published State:

| publish_run_id | live_master_page_id | live_master_version | live_master_hash | baseline_schema_version | baseline_schema_hash | published_module_versions | published_at | result |
|---|---|---:|---|---:|---|---|---|---|

Module Source State:

| module_key | page_id | last_synced_version | last_synced_hash | last_synced_at | last_run_id |
|---|---|---:|---|---|---|

After a successful Draft write, update Draft Proposal State and Module Source State. After a successful Live Master publish, update Published State. Do not update Published State just because Draft was written.

## Live Master Rule

Live Master is not part of normal automated sync. Only write Live Master when the PJ explicitly asks after reviewing Draft Master Copy.

If asked to publish live:

- Re-read Draft, Live Master, Baseline, and Action Log immediately before writing.
- Block publish if Live Master version/hash differs from the Draft proposal base.
- Block publish if Baseline schema version/hash differs from the Draft proposal schema.
- Copy only approved Proposed Master applicable content.
- Preserve all human-added or unmanaged Live Master content.
- Apply proposed removals only if PJ explicitly approved those removals in the same request.
- Exclude `[EXCLUDED BY SCHEMA] Module End Notes` unless PJ explicitly approves inclusion and the Baseline target/schema is clear.
- Summarize published additions, updates, skipped removals, and preserved human notes.
