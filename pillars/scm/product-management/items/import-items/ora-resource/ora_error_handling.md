# Error Handling

| Situation | Required action |
|---|---|
| Source is unreadable, ambiguous, or missing required values | Stop before CSV generation and request the smallest clarification. |
| Current public template cannot be fetched or differs from the local contract | Stop as template drift; do not use a stale layout. |
| CSV or ZIP validation fails | Discard the incomplete transient artifact and regenerate after correcting the local issue. |
| ZIP changes after validation | Restart template-driven validation and produce new handoff evidence. |
| Handoff metadata is incomplete | Stop before handing the ZIP to `core.data-loader`. |
| A later Item Import reports Succeeded but creates no items and logs no `EGP_IMPORT_ERRORS`; interface rows stay at `PROCESS_STATUS` 0 with `REQUEST_ID` set, plus item-level rows without organization at −1 | Classify it as a data-quality matching hold, not a package defect. Do not rebuild or resubmit the package. Route the items through a registered item batch and resolve the In Progress match actions (`ora_design_advisory.md`). |
| Item batch In Progress rows show match action Confirm against an existing item number for items that must be new | Stop. Set New Item, Save, and re-check the item numbers before processing. Processing Confirm updates the existing item. |
| A structure import reports "specified item does not exist" for parents whose item load reported success | Check the item batch for held matches before debugging the structure package. |

Classify these failures as input validation, catalog mismatch, or artifact
persistence. Classify a data-quality matching hold as an approval blocker: the
items wait for a match decision. This skill does not perform authorization, upload, import,
monitoring, ESS, or Fusion API operations; `core.data-loader` handles any
later execution failure in its own workflow.

## Task Nodes

| Rule ID | Node Type | Sequence | Action Type | Transitions | Prerequisites | On Failure | Applies When |
|---|---|---|---|---|---|---|---|
| ITEM_IMPORT.HANDOFF_GATE | decision | 3 | execution_gate | ready->ITEM_IMPORT.READY_FOR_CORE_DATA_LOADER;blocked->ITEM_IMPORT.FAILED | ITEM_IMPORT.CONFIRM_LOADER_ROUTE | stop | Immediately before handing the unchanged validated ZIP to `core.data-loader`. |
| ITEM_IMPORT.READY_FOR_CORE_DATA_LOADER | terminal | 4 | terminal | none | ITEM_IMPORT.HANDOFF_GATE | continue | Local validation and immutable handoff evidence are complete. |
| ITEM_IMPORT.FAILED | terminal | 5 | terminal | none | none | continue | Any local validation, template, or artifact-persistence failure occurs. |