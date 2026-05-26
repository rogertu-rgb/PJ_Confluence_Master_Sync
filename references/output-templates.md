# Output Templates

Use these formats for concise user-facing responses and logs.

## Baseline Understanding

```text
Baseline Understanding
- Status:
- Project pages found:
- Modules found:
- Content elements found:
- Master placement found:
- Sync state found:
- Ambiguities:
- Missing required items:
- Recommended next action:
```

## Merge Plan

```text
Merge Plan
- Status: safe_to_write / needs_review / blocked
- Project:
- Baseline:
- Live Master base:
- Draft Master Copy:
- Schema changed since last Draft / publish:
- Module pages to collect:
- Changed modules:
- Unchanged modules:
- Missing or unreadable modules:
- Placement plan:
- Proposed Master body:
- Module-end out-of-schema notes:
- Change ledger index:
- Proposed removals:
- Human-preserved content:
- Conflicts:
- Recommended next action:
```

## Draft Proposal Outline

```text
Draft Merge Proposal
- Status:
- Draft run ID:
- Live Master base version/hash:
- Baseline schema version/hash:
- Module versions:
- Schema change impact:

Change Ledger
| target_path | operation | module_key | element_key | previous_master_summary | proposed_summary | reason | schema_status | publish_action |

Proposed Master Content
Full current Live Master body first, with inline labels at the correct target placements:
[NEW] / [UPDATED] / <s>[REMOVED - PROPOSED]</s> / [HUMAN PRESERVED]

Module End Notes For Out-of-Schema Content
[EXCLUDED BY SCHEMA] content appears at the end of the related module section and will not be included unless PJ updates schema/mapping or explicitly approves inclusion.

Proposed Removals
<s>[REMOVED - PROPOSED]</s> content remains visible in the inherited Master body and will be removed only if PJ explicitly approves removal.

Conflicts And PJ Decisions Needed

Publish Command Guidance
```

## Final Run Result

```text
Confluence Master Sync Result
- Status:
- Project:
- Pages used:
- Live Master base:
- Baseline schema:
- Modules checked:
- Modules changed:
- Draft Master Copy:
- Proposed Master body:
- Module-end out-of-schema notes:
- Change ledger index:
- Proposed removals:
- Human-preserved content:
- Conflicts:
- Missing / low confidence:
- Action Log:
- Next PJ action:
```

## Action Log Row

| run_id | log_time | actor | mode | action_type | project | target_page | source_page_ids | source_versions | draft_before_version | draft_after_version | live_master_base_version | baseline_schema_version | draft_run_id | publish_run_id | summary | result | status_reason | next_action |
|---|---|---|---|---|---|---|---|---|---:|---:|---:|---:|---|---|---|---|---|---|
| sync-YYYYMMDD-HHMMSS | YYYY-MM-DD HH:mm | Codex | manual_browser | draft_proposal | Project Name | Draft Master Copy | 123,456 | M1:v2,M2:v1 | 4 | 5 | 12 | 8 | draft-YYYYMMDD-HHMMSS |  | Checked N modules, changed M, wrote Draft proposal. | needs_review | held_out_content | PJ review Draft proposal. |

Minimum acceptable log row when the full schema is unavailable:

| log_time | actor | action_type | project | target_page | summary | result | next_action |
|---|---|---|---|---|---|---|---|
| YYYY-MM-DD HH:mm | Codex | draft_merge | Project Name | Draft Master Copy | Checked N modules, changed M, wrote draft output. | success / needs_review / blocked | PJ review Draft Master Copy. |

## Live Publish Result

```text
Confluence Master Sync Publish Result
- Status:
- Project:
- Draft proposal:
- Live Master before:
- Live Master after:
- Baseline schema:
- Published additions:
- Published updates:
- Approved removals applied:
- Proposed removals skipped:
- Human/unmanaged content preserved:
- Blocked reason:
- Action Log:
- Next PJ action:
```

## Scheduled Run Blocked Message

```text
Confluence Master Sync Result
- Status: blocked
- Project:
- Pages used:
- Modules checked: not run
- Modules changed: not run
- Draft Master Copy: not written
- Conflicts:
- Missing / low confidence: scheduled Confluence auth is not available
- Action Log: not written / returned row only
- Next PJ action: provide approved non-interactive Confluence auth method or run manually from an authenticated browser session.
```
