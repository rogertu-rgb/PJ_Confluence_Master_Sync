# Baseline Mapping Reference

Use this when a PJ-provided Baseline is not in a fixed template.

## Minimal Model

| Area | Required meaning |
|---|---|
| Project pages | Baseline, Master File, Draft Master Copy, Action Log |
| Modules | module name, source page link, include flag, order |
| Content elements | element name/key, required flag, source description |
| Master/Draft structure | layer names and how module/element/content blocks appear in Master and Draft |
| Master placement | target section/path in Draft/Master |
| Sync state | latest Draft proposal state, latest published state, module page versions/hashes when available |

## Minimal Baseline Contract

The Baseline can look different by project, but it must let the skill identify these concepts:

| Concept | Required? | Preferred fields |
|---|---:|---|
| Project pages | Yes | page_role, title, page_id or link |
| Module registry | Yes | module_key, module_name, include, order, module_page_id or link |
| Content elements | Yes for writeback | element_key/name, required flag, source description |
| Master/Draft structure | Recommended | layer_id, layer_name, page_heading_pattern, maps_to |
| Master placement | Yes for writeback | module_key or element_key, target path/section |
| Latest Draft proposal state | No for first run, yes after first Draft | draft_run_id, draft_version, live_master_base_version/hash, baseline_schema_version/hash |
| Latest published state | No before first publish, yes after first publish | publish_run_id, live_master_version/hash, baseline_schema_version/hash |
| Module source state | No for first run, yes after first Draft | module_key, page_id, last_synced_version, last_synced_hash/time |

If required concepts are present but field names differ, normalize them. If a required concept is absent, return `needs_review` for understanding and `blocked` for writeback.

## Flexible Wording

| PJ wording | Normalize to |
|---|---|
| Component / Domain / Workstream / Track | module name |
| Owner Doc / Source Page / Module Page | module source page |
| Field / Requirement / Workflow item / Output | content element |
| Master Section / Output Section / Target Path | master placement |
| Last Sync / Baseline Version / Page Version | sync state |

## Mapping Logic

- Prefer explicit links and page IDs over titles.
- Prefer stable labels, element keys, table columns, anchors, and target paths over heading depth.
- Do not require all projects to use the same number of layers.
- If a Baseline adds a layer between parent and child, keep mapping valid when the element key and target path remain understandable.
- If an added layer changes meaning or creates duplicate targets, mark `needs_review`.
- If source and target cannot be connected with reasonable confidence, mark `blocked` for writeback.

## Structure Map

Use a simple structure map when PJ wants to reason about the Master/Draft layers:

| layer_id | layer_name | Master pattern | Draft behavior | Baseline mapping |
|---|---|---|---|---|
| L1 | Project / Master root | page title or project heading | inherited unchanged | project_pages.master |
| L2 | Module section | module_key + module_name heading | inherited; module-level notes preserved | modules.module_key |
| L3 | Schema element section | element heading/key | inline [NEW]/[UPDATED]/strikethrough changes | content_elements.element_key + master_placement.target_path |
| L4 | Content block | paragraph/table/list under element | proposed content body | extracted module content |
| L5 | Module end placeholder | optional end-of-module area | [EXCLUDED BY SCHEMA] Module End Notes | outside schema until PJ updates Baseline |

The Draft should mimic the Master structure first. Schema-out content belongs in L5 placeholders near the related module, not in an unrelated page-level bucket.

## Latest State

Prefer canonical latest-state tables instead of appending a new state table each run.

Draft proposal state updates after Draft Master Copy is written:

| draft_run_id | draft_page_id | draft_version | live_master_base_version | live_master_base_hash | baseline_schema_version | baseline_schema_hash | module_versions | generated_at | result |
|---|---|---:|---:|---|---:|---|---|---|---|

Published state updates only after Live Master is successfully published:

| publish_run_id | live_master_page_id | live_master_version | live_master_hash | baseline_schema_version | baseline_schema_hash | published_module_versions | published_at | result |
|---|---|---:|---|---:|---|---|---|---|

Module source state tracks module versions/hashes used for proposals:

| module_key | page_id | last_synced_version | last_synced_hash | last_synced_at | last_run_id |
|---|---|---:|---|---|---|

Action Log preserves history; Baseline preserves the latest Draft proposal state, latest published state, and latest module source state separately.

## Required Baseline Understanding Output

```text
Baseline Understanding
- Status: confirmed / needs_review / blocked
- Project pages found:
- Modules found:
- Content elements found:
- Master placement found:
- Sync state found:
- Ambiguities:
- Missing required items:
- Recommended next action:
```

## Status Rules

Use `confirmed` only when all required pages and included module links are found and module-to-target placement is clear.

Use `needs_review` when a mapping is plausible but PJ confirmation is needed.

Use `blocked` when Baseline cannot be read, Draft Master Copy cannot be identified, or an included module has no readable page.

## Common Ambiguities

- Multiple pages look like Master File or Draft Master Copy.
- Module is listed but no source page is linked.
- Same module appears under two names.
- Two modules map to the same target.
- A content element exists but has no target.
- Baseline wording is project-specific and could mean several things.
- Baseline schema changed after the last Draft proposal or Live Master publish, requiring full revalidation instead of changed-module-only validation.
