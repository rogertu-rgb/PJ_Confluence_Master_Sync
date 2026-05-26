# PJ Confluence Master Sync

Codex skill for maintaining a Confluence-only PJ project workspace where module pages are read-only sources and automated output is written only to a Draft Master Copy.

## What This Skill Does

- Understands a PJ-owned Baseline instead of assuming one universal project schema.
- Discovers Project Folder, Master File, Draft Master Copy, Action Log, Baseline, and module pages.
- Reads module pages as source material and detects changed/new/unreadable modules.
- Builds a Draft Master Copy proposal with inline labels such as `[NEW]`, `[UPDATED]`, `[REMOVED - PROPOSED]`, `[EXCLUDED BY SCHEMA]`, `[CONFLICT]`, and `[HUMAN PRESERVED]`.
- Preserves Live Master content until the PJ explicitly approves publish.
- Records run details in Action Log when available.

## Install For Codex

Clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/rogertu-rgb/PJ_Confluence_Master_Sync.git ~/.codex/skills/pj-confluence-master-sync
```

Restart Codex or reload skills, then invoke:

```text
Use $pj-confluence-master-sync to inspect this Confluence workspace and return Baseline Understanding.
```

## Skill Layout

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── baseline-mapping.md
    ├── confluence-workflow.md
    ├── log-query.md
    ├── output-templates.md
    └── proposal-versioning.md
```

## Access Modes

The skill can run manually when Codex has an authenticated browser session, approved REST credentials, or a Confluence connector. Scheduled runs require non-interactive access through an approved REST credential or connector.

Never put credentials, cookies, tokens, or machine-specific setup into the skill, Baseline, Action Log, or GitHub repository.

## Safe Write Rules

- Module pages are read-only sources.
- Draft Master Copy is the only automated write target.
- Live Master is publish-only after explicit PJ review and approval.
- Schema-driven removals are shown as proposed removals first.
- Content outside the Baseline schema is held as module-end notes until the PJ maps or approves it.

## Main Workflow

1. Read the Project Folder and Baseline.
2. Return Baseline Understanding.
3. Ask the PJ to confirm ambiguous mappings or missing core pages.
4. Detect module changes and schema changes.
5. Build a Master-shaped Draft proposal.
6. Write only to Draft Master Copy when safe.
7. Append Action Log and update Draft proposal state when available.
8. Publish to Live Master only after explicit PJ approval.
