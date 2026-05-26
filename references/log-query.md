# Action Log Query Reference

Use this when the PJ asks what happened before, whether a sync ran, why a run was blocked, or what changed in the last merge.

## Queryable Fields

Prefer the expanded Action Log schema when present:

| Field | Use |
|---|---|
| run_id | Find all records for one run. |
| log_time | Filter by date/time or latest run. |
| actor | Identify who or what triggered the action. |
| mode | Distinguish manual_browser, rest_token, connector, scheduled. |
| action_type | Filter readiness_check, config_understanding, update_check, module_collect, draft_merge, sync_state_update, blocked. |
| project | Confirm the project workspace. |
| target_page | Identify Draft, Baseline, Action Log, or affected page. |
| source_page_ids | Identify module pages used. |
| source_versions | See source versions read during the run. |
| draft_before_version / draft_after_version | Audit Draft writeback. |
| result | Find success, needs_review, blocked, failed. |
| status_reason | Explain why a status was returned. |
| next_action | Show what the PJ should do next. |

If only the minimum schema exists, query the available fields and state which fields are missing.

## Common User Prompts

```text
Use $pj-confluence-master-sync to query the Action Log for this project: <Project Folder URL>. What was the latest run and what should I do next?
```

```text
Use $pj-confluence-master-sync to query this Action Log: <Action Log URL>. Show only blocked or needs_review runs.
```

```text
Use $pj-confluence-master-sync to query the Action Log for module M4. Did it change in the latest merge?
```

```text
Use $pj-confluence-master-sync to query run_id sync-YYYYMMDD-HHMMSS and summarize pages read, Draft version change, conflicts, and next action.
```

## Query Response

```text
Action Log Query Result
- Status:
- Project:
- Filter used:
- Matching runs:
- Latest relevant run:
- Pages affected:
- Result / reason:
- Next PJ action:
- Missing log fields:
```

## Rules

- Do not modify Confluence pages in log query mode.
- If Action Log cannot be found, read Baseline or Project Folder to find it.
- If multiple Action Logs are found, ask PJ to choose or return `needs_review`.
- If log fields are missing, still answer from available fields and recommend an Action Log schema upgrade.
