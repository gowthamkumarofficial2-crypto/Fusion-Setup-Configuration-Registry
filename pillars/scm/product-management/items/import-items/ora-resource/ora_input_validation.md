# Input Validation

Validate customer item data or a prebuilt Item Import FBDI ZIP against the
current public Oracle Item Import template before immutable handoff to
`core.data-loader`.

| Requirement | Validation gate | Failure action |
|---|---|---|
| Source or ZIP | Readable customer source or a readable ZIP is present | Stop before mapping. |
| Current template | Current public `ItemImportTemplate.xlsm`, URLs, retrieval time, digest, and sheet sequence are recorded | Stop; never use a remembered layout. |
| Mapping | Every required source fact maps to the current template or has an explicit documented default | Stop for missing or ambiguous values. |
| CSV and ZIP shape | Headerless CSVs, exact filenames, ordered root-level members, safe paths, and exact field counts | Stop and regenerate. |
| Data validity | Current-template types, constraints, and references pass | Stop and correct the source. |
| Batch reference | Exactly one form is used in every populated member: one numeric `BATCH_ID`, or one registered item batch Number in `BATCH_NUMBER` with `BATCH_ID` blank (resolved by Oracle's control file at load) | Stop; never mix both forms in one package. |
| Data-quality matching risk | Source system and its data-quality options are recorded, look-alike items (same description as an existing master-org item) are listed, and the planned outcome is confirmed by the user: a registered batch with New Item confirmation, or matching deselected on the batch. See `ora_design_advisory.md`. | Stop before handoff until the user confirms the plan. |
| Immutable evidence | ZIP digest, member manifest, row counts, template provenance, and mapping preview are present | Stop before handoff. |
| Data Loader route | Handoff identifies registered FBDI Data Loader business object `item` | Stop and correct the task configuration. |

## Task Nodes

| Rule ID | Node Type | Sequence | Action Type | Transitions | Prerequisites | On Failure | Applies When |
|---|---|---|---|---|---|---|---|
| ITEM_IMPORT.VALIDATE_FBDI | step | 1 | validate_input | valid->ITEM_IMPORT.CONFIRM_LOADER_ROUTE;invalid->ITEM_IMPORT.FAILED | none | stop | Validate source, current template, package, and immutable evidence before handoff. |