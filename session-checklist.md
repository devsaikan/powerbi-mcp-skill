# Power BI MCP - Session Checklist and Documentation Templates

Reference file for SKILL.md. Use at session start and session close.

---

## Session Startup Checklist

Run before any Tier 2 or Tier 3 action. Present this to the user and
wait for responses before proceeding with write operations.

SESSION SAFETY CHECKLIST

1. Last manual save:
   When was the model last saved in Power BI Desktop?

2. Backup exists:
   Has a TMDL export been taken, or is the .pbix in version control? (YES / NO)

3. Rollback plan:
   If something goes wrong, how will you restore?
   Options: revert .pbix / restore TMDL / git checkout

4. Session scope - what are we doing today?
   - Read-only (schema review, documentation, null analysis)
   - DAX authoring (create or modify measures only)
   - Structural changes (tables, relationships, roles)
   - Destructive operations permitted (deletes allowed)

5. Model size:
   Approximately how many existing measures does the model have?
   This determines the context strategy.

If the user skips this, prompt gently:
"Before we make any changes - when was the model last saved, and is there a TMDL export or backup available?"

Do not proceed to Tier 2 or Tier 3 without at minimum items 1 and 2 confirmed.

---

## Session Close - Measures Catalog

Auto-generate at the end of any session where measures were created or modified.

Measures Catalog - [Model Name]
Generated: [Date] | Session: [brief description]

Measure | Folder | Format | Purpose | Key Dependencies
[name]  | [folder] | [format string] | [one-line business description] | [tables, columns, measures referenced]

Include every measure touched in the session - created, modified, or reviewed.

---

## Session Close - Change Log

Auto-generate at the end of any session where writes occurred.

Session Change Log - [Date]
Model: [Model Name] | Analyst: [if known] | Duration: [approx]

Created:
- [Object Name] ([type: measure / column / table / relationship])
  [Brief description of what it does and why it was created]

Modified:
- [Object Name] - [what changed and why]
  Before: [previous definition or state if known]
  After: [new definition]

Deleted:
- [Object Name] - [why removed]
  Confirmed by user: [timestamp or "confirmed verbally"]
  Backup taken: YES / NO

Proposed (not yet applied):
- [Object Name] - [what was proposed, why it is pending]
  Status: Awaiting user decision

M Code Delivered (manual application required):
- [Transformation Name] - [table, what it does]
  Status: Provided to user for manual paste in Power Query Editor

Issues Encountered:
- [Any DAX errors, relationship ambiguities, missing tables, or
  data quality issues discovered during the session]

---

## Pre-Publish Checklist

Run before the user closes Power BI Desktop after a write session.

PRE-PUBLISH CHECKLIST

- All debug measures set to IsHidden = true
- All measures have a display folder assigned (none in root)
- All new relationships validated (no ambiguous paths)
- TMDL export taken (post-session backup)
- Measures catalog generated and saved
- Change log generated and saved
- Model refreshed successfully with new measures
- Sample visual tested for each new measure

---

## Complexity Threshold Reference

Under 50 measures:    standard session, full context
50 to 150 measures:   export TMDL first, work from file reference
150 to 300 measures:  split into domain sessions (Financial, Operational, etc.)
Over 300 measures:    mandatory TMDL export and full dependency map before any changes

For models with 150 or more measures, always run a read-only dependency scan first:
"Which measures reference [table being modified]?"
"Which measures will be affected if [column] is renamed?"
