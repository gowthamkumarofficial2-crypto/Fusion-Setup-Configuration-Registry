# Item Import FBDI Processing

## Purpose

Define the template-driven mapping, packaging, and validation contract for Item
Import. This skill prepares one immutable archive and hands it unchanged to
`core.data-loader`; it never executes a Fusion transport or import.

## Current Public Template Authority

For every run, fetch `ItemImportTemplate.xlsm` from the current public Oracle
File-Based Data Import Item page. Record the page URL, workbook URL, retrieval
time, SHA-256 digest, and sheet sequence. The workbook and its linked control
files are the sole schema authority for that run.

Read each populated sheet's ordered export columns, technical names, types,
required markers, validations, and allowed values. Preserve its complete column
order, trailing empty fields, encoding, delimiter, quoting, and null
conventions. For `EGP_SYSTEM_ITEMS_INTERFACE`, exclude only the workbook-only
`Name` metadata column; the first headerless CSV field is `TRANSACTION_TYPE`.

Stop on template drift: a changed sheet sequence, output filename, or export
contract requires updating this task skill before creating a package.

## Package Contract

Create headerless root-level CSV members only for populated sheets. Use the
current template's prescribed filenames and preserve this relative order:

1. `EgpSystemItemsInterface.csv`
2. `EgpItemRevisionsInterface.csv`
3. `EgpItemCategoriesInterface.csv`
4. `EgpItemRelationshipsIntf.csv`
5. `EgoItemAssociationsIntf.csv`
6. `EgoItemIntfEffb.csv`
7. `EgoItemIntfEfftl.csv`
8. `EgoItemRevisionIntfEffb.csv`
9. `EgoItemRevisionIntfEfftl.csv`
10. `EgoItemSupplierIntfEffb.csv`
11. `EgoItemSupplierIntfEfftl.csv`
12. `EgoStyleVariantAttrvsIntf.csv`
13. `EgpTradingPartnerItemsIntf.csv`
14. `EgpItemAttachmentsIntf.csv`
15. `EgpMpnSpnRelLinkageInt.csv`
16. `AcaItemCostInterface.csv`
17. `EgpItemOrgAssociations.csv`

Use one numeric Batch ID consistently in every populated member at its template
position; in `EgpSystemItemsInterface.csv` it is the second CSV field.

For a registered item batch (Manage Item Batches), instead populate
`BATCH_NUMBER` with the batch Number in every populated member and leave
`BATCH_ID` blank. In `EgpSystemItemsInterface.csv`, `BATCH_NUMBER` is the third
CSV field. Oracle's control file resolves `BATCH_ID` from
`EGI_IMPORT_BATCHES_B` at load time; it uses `-1` when the Number is not
found. Never mix the two forms in one package. See `ora_design_advisory.md`
for when a registered batch is required.

## Validation and Immutable Handoff

Reopen the completed ZIP and fail closed unless it has safe root-level members,
no duplicates or unexpected files, exact template-prescribed names and order,
headerless CSV rows with exact field counts, valid values, coherent
cross-file references, one batch reference (Batch ID or registered batch
Number), and a SHA-256-bound preview.

Hand that exact archive, without copying or changing it, to `core.data-loader`
with task context and registered FBDI Data Loader business object `item`.
Supply template provenance, ZIP digest, member manifest, row counts, mapping
preview, and batch reference as read-only evidence. For a registered batch,
include both the batch Number and its internal `BATCH_ID`, which the Item
Import *Batch Id* parameter requires.

Do not upload the ZIP, call an ESS job, initiate or monitor an import, query a
Fusion execution endpoint, construct execution parameters, or perform UCM
basename checks. `core.data-loader` owns all later authorization, transport,
submission, and monitoring decisions.