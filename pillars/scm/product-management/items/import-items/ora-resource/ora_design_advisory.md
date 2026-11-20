# Design Advisory

## Product Hub Data-Quality Matching on Item Import

Item Import can end **Succeeded** without creating items and without writing
any error. This happens when Product Hub data-quality matching holds the
incoming items for a match decision. Assess this risk before building the
package and plan for it.

### Oracle-documented behavior

Verify each point against the latest release documentation before relying on
it. The points below were checked against 26D.

- **Where the options live.** Data-quality options are defined on the source
  (spoke) system and can be overridden per item batch:
  - Enable data quality matching
  - Confirm single matches
  - Confirm unmatched as new item

  Oracle Product Hub itself is the spoke system *Product Information
  Management Data Hub*.
  ([Available Options to Check Data Quality for Item Batches](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/26d/fapim/available-options-to-check-data-quality-for-item-batches.html))
- **What happens on data load.** For items created or updated from the Product
  Data Hub and imported during data load:
  - items with no match are imported;
  - items with a single or multiple matches stay in the item batch's
    **In Progress** tab with match action *Unconfirm*.
  (same page)
- **Match actions.** *Confirm* matches the source item to an existing Product
  Hub item. *New Item* creates a new item during import. *Exclude* and
  *Ignore* leave the item out.
  ([Unconfirmed Items in Item Batches](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/26d/fapim/unconfirmed-items-in-item-batches.html),
  [Confirmed Items in Item Batches](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/26d/fapim/confirmed-items-in-item-batches.html))
- **Registered item batches.** An item batch definition inherits the source
  system's options, can override them, and accepts items by FBDI.
  ([Item Batches](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/26d/fapim/item-batches.html))
- **Batch Number vs Batch ID.** A batch's Number and its internal Batch ID are
  different values.
  ([Get and Import Item Batches](https://docs.oracle.com/en/cloud/saas/supply-chain-and-manufacturing/26d/fasrp/use_case_get_import_item_batches.html))

### Field-verified behavior (26D, sanitized implementation evidence)

Source-system options in this case: matching on, confirm single matches off.
New items whose descriptions duplicated existing master-org items gave this
signature:

1. Item Import ended Succeeded, created no items, and wrote no
   `EGP_IMPORT_ERRORS` rows.
2. In `EGP_SYSTEM_ITEMS_INTERFACE`, the loaded rows kept `PROCESS_STATUS` 0
   with `REQUEST_ID` stamped.
3. One extra row per item appeared, with no organization, at
   `PROCESS_STATUS` −1. Oracle does not document these status values; treat
   them as a signature, not a meaning.

With an unregistered (made-up) numeric Batch ID, Manage Item Batches has no
page for the batch, so the held items stayed invisible. Loaded through a
registered item batch, the same items appeared In Progress. Changing them to
**New Item**, then Save, Process Data and import, created them.

A follow-on structure import for those parents failed with "specified item
does not exist". That error is a consequence of the held items, not a separate
cause.

### Preparation guidance

1. **Record the source system's options.** Note the `SOURCE_SYSTEM_CODE` the
   package uses and ask for that system's data-quality options as evidence
   (for example `EGI_IMPORT_OPTION_SETS`: `MATCH_ON_DATA_LOAD`,
   `CONFIRM_SINGLE_MATCH`, `CONFIRM_UNMATCHED_NEW`, `IMPORT_ON_DATA_LOAD`).
   Flag the risk when matching is on and confirm single matches is off.
2. **Flag look-alike items.** List new items whose description equals an
   existing item's description in the target master org
   (`EGP_SYSTEM_ITEMS_TL` joined to `EGP_SYSTEM_ITEMS_B`). This is a
   heuristic: the actual match rules are configuration-specific.
3. **Prefer a registered item batch.** Populate `BATCH_NUMBER` with the batch
   Number and leave `BATCH_ID` blank. Oracle's item control file resolves
   `BATCH_ID` from `EGI_IMPORT_BATCHES_B` at load time.
   - Record both the Number and the internal `BATCH_ID` in the handoff
     evidence.
   - The Item Import *Batch Id* parameter takes the internal `BATCH_ID`.
4. **Plan the outcome for intentional look-alike items.** Either deselect
   *Enable data quality matching* on the batch (documented per batch; not
   field-tested here), or plan the **New Item** confirmation.
5. **Never process a Confirm row whose item is an existing item number.**
   Processing it updates that existing item instead of creating the new one.
