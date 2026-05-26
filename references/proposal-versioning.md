# Draft Proposal And Versioning Reference

Use this when generating Draft Master Copy or publishing an approved Draft into Live Master.

## Principle

Draft Master Copy is the current merge proposal. It is not the source of truth and it is not the only version-control layer.

Version control comes from:

- Confluence page versions for Live Master, Draft Master Copy, Baseline, modules, and Action Log.
- Action Log history for every run.
- Baseline state tables for current Draft proposal state and last published Live Master state.

The Draft page should always exist for a project, but its managed section can be replaced each run. Previous Draft proposals remain traceable through Confluence page history and Action Log run IDs.

## State Tables In Baseline

Keep Draft proposal state separate from published state.

### Latest Draft Proposal State

| draft_run_id | draft_page_id | draft_version | live_master_base_version | live_master_base_hash | baseline_schema_version | baseline_schema_hash | module_versions | generated_at | result |
|---|---|---:|---:|---|---:|---|---|---|---|

Update after writing Draft Master Copy.

### Latest Published State

| publish_run_id | live_master_page_id | live_master_version | live_master_hash | baseline_schema_version | baseline_schema_hash | published_module_versions | published_at | result |
|---|---|---:|---|---:|---|---|---|---|

Update only after Live Master is successfully published.

## Schema Change Rule

Before every Draft proposal:

1. Read current Baseline schema version/hash.
2. Compare it with the latest Draft proposal state and latest published state.
3. If schema changed, revalidate all included modules and all previously published managed elements.
4. If schema did not change, validate changed modules plus dependent placements.

Schema changes can make content newly eligible or newly ineligible. The Draft must show both directions:

- `newly_eligible`: content was previously held out but now maps to schema.
- `newly_ineligible`: content was previously eligible or published but no longer maps to schema.
- `removed_proposed`: content should be removed from the managed Master output if PJ approves.
- `excluded_by_schema`: module content remains outside schema and is not placed into Proposed Master.

## Draft Proposal Format

The managed Draft section must use visible markers:

```html
<p><strong>PJ_CONFLUENCE_MASTER_SYNC_START</strong></p>
...
<p><strong>PJ_CONFLUENCE_MASTER_SYNC_END</strong></p>
```

Inside the managed section, use this order:

1. Proposed Master Content
2. Module End Notes For Out-of-Schema Content, placed at the end of each related module section
3. Compact Sync Metadata And Version Guardrails
4. Change Ledger Index
5. Conflicts And PJ Decisions Needed
6. Publish Command Guidance

Do not place an old Draft scaffold, placeholder module list, or "Draft Output Area" above the inherited Master-shaped proposal. The Draft should read like the Master page with inline review marks.

## Master Inheritance Rule

The Proposed Master Content is the primary review surface.

It must:

- Start from the complete current Live Master structure and content.
- Preserve unchanged Master content in place.
- Insert schema-eligible module content at the correct Baseline target placement.
- Show changes inline where PJ expects them to land, not only in a separate ledger table.
- Keep module order, section order, and human notes visible unless there is an explicit reason to mark a change.

The Change Ledger and metadata are only supporting sections after the Proposed Master body. They should help PJ navigate and publish safely, not replace or visually dominate the inline review.

## Inline Edit Marking

For each target placement:

- New content: insert at the target location with `[NEW]`.
- Updated content: show `[UPDATED]`, the previous Master content summary, and the proposed replacement near the original location.
- Deleted or newly ineligible content: keep it at the original Master location and render it with strikethrough, for example `<s>[REMOVED - PROPOSED] old content</s>`.
- Unchanged human or unmanaged content: keep it in place and optionally label as `[HUMAN PRESERVED]` when relevant.
- Conflicting content: leave the current Master content in place and add `[CONFLICT]` note at the relevant location.

Do not remove content from the Draft proposal body merely because schema changed. Removal is only proposed visually until PJ approves publish.

## Module End Out-of-Schema Notes

Content outside the current Baseline schema should remain near its module, not disappear into a detached table.

For each module, after its schema-managed target content in the Proposed Master body, add a module-end area when needed:

```text
[EXCLUDED BY SCHEMA] Module End Notes - <module_key>
- element/source:
- content summary:
- reason:
- PJ decision needed: add to schema / map target / keep as module note / ignore
```

These notes are visible in Draft for review, but are not publishable managed Master content unless PJ explicitly approves inclusion and the Baseline schema/target placement is clear.

## Version Guardrails

Always include:

- `draft_run_id`
- `live_master_page_id`
- `live_master_base_version`
- `live_master_base_hash`
- `baseline_page_id`
- `baseline_schema_version`
- `baseline_schema_hash`
- module page IDs and versions
- whether schema changed since last Draft proposal
- whether schema changed since last published state

If any guardrail is missing, status should be `needs_review` or `blocked` depending on write risk.

## Change Ledger

Every affected target should appear in a table:

| target_path | operation | module_key | element_key | previous_master_summary | proposed_summary | reason | schema_status | publish_action |
|---|---|---|---|---|---|---|---|---|

Allowed operations:

- `new`: content will be added to Proposed Master.
- `updated`: existing managed content will change.
- `removed_proposed`: existing managed content is no longer schema-eligible and is proposed for removal.
- `unchanged`: content remains the same.
- `excluded_by_schema`: module content is held out because it is not in schema or has no target.
- `conflict`: multiple inputs or unclear mappings require PJ decision.
- `human_preserved`: current Live Master content is unmanaged or human-added and will be preserved.

## Formatting Constraints

Use stable text labels so the result is readable even when Confluence macros are unavailable:

- `[NEW]`
- `[UPDATED]`
- `[REMOVED - PROPOSED]`
- `[EXCLUDED BY SCHEMA]`
- `[CONFLICT]`
- `[HUMAN PRESERVED]`

Use Confluence panels, status macros, background colors, or tables only as decoration. The text labels must remain.

For each changed target, show:

- previous Master summary
- proposed content summary
- source module and element
- schema reason
- publish action needed

## Human Note Protection

Do not erase human-added notes.

Rules:

- Preserve Live Master content outside managed sync sections.
- If a target is not clearly managed by the skill, classify it as `human_preserved` or `needs_review`.
- If schema says content is no longer eligible, show it as `removed_proposed` in Draft first.
- Publish can remove content only when it is in a managed sync section and PJ explicitly approves that removal in the same publish request.
- If approval is unclear, keep the content and list it as `not_removed`.

## Held-out Content

Held-out Content contains module content that is useful but not placed into Proposed Master:

- outside Baseline schema
- missing target placement
- conflicting with another module
- low-confidence extraction
- newly ineligible after schema change

For each held-out item, include:

- module key
- element key or source locator
- content summary
- reason held out
- suggested PJ decision: add to schema, map target, keep as module note, or ignore

Prefer rendering held-out items as Module End Notes inside the inherited Proposed Master body. A separate held-out table may be included as a summary index, but it should not be the only place where out-of-schema content appears.

## Publish Rule

Before Live Master publish:

1. Re-read Draft, Live Master, Baseline, and Action Log.
2. Check Draft `live_master_base_version/hash` against current Live Master.
3. Check Draft `baseline_schema_version/hash` against current Baseline.
4. If either changed, block publish and regenerate/rebase Draft.
5. Copy only approved Proposed Master applicable content.
6. Preserve human/unmanaged content.
7. Exclude `[EXCLUDED BY SCHEMA] Module End Notes` unless PJ explicitly approves inclusion and the Baseline target/schema is clear.
8. Apply `removed_proposed` only if PJ explicitly approved removals.
9. Update published state only after Live Master write succeeds.
10. Log publish result and any skipped removals or skipped module-end notes.
