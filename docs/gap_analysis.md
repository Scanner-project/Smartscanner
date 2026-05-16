# Gap Analysis — PRD vs Implementation vs Data Dictionary

Summary of notable discrepancies and recommended immediate fixes.

## Discrepancies
- `total` vs `totalAmount`:
  - `Receipt.toJson()` uses the key `total` for serialization.
  - Firestore publish in `ReceiptRepository._publishReceiptToFirestore` uses `totalAmount`.
  - Risk: downstream Firestore consumers may expect `total` or `totalAmount` inconsistently.

- `items` missing from processed receipts:
  - `ReceiptProcessingService.processReceiptImage` returns receipts with `items: []` and `category: Other`.
  - The UI and downstream analytics may expect itemized data.

- Firestore doc id vs in-memory `Receipt.id`:
  - Firestore `.add()` creates a server doc id but the local `Receipt.id` (milliseconds) is not synchronized back to the Firestore document.

- Gallery ↔ Receipt linking:
  - `GalleryImage.linkedReceiptId` exists; `Receipt.galleryImageId` exists; the publish flow does not populate/record these consistently.

## Recommended immediate fixes (prioritized)
1. Unify amount key: choose `total` or `totalAmount`. Update `_publishReceiptToFirestore` to use the same key as `Receipt.toJson()` (or vice versa) and add unit tests.
2. Decide on itemization strategy: either extend `ReceiptProcessingService` to extract items or annotate that item extraction is out-of-scope. If extracting, implement heuristics + tests.
3. When publishing to Firestore, include the local `Receipt.id` (or map Firestore doc id back to local state) to enable reliable cross-referencing.
4. Define and implement a consistent linking flow between `GalleryImage` and `Receipt` (which field is authoritative) and persist it in both local storage and Firestore if intended.

## Suggested quick follow-ups (small PRs)
- Add unit tests for serialization/deserialization of `Receipt` and `GalleryImage`.
- Add an integration test that simulates `processReceiptImage` and verifies `addReceipt` → local storage + Firestore call.
- Add a short note to `PRD.md` specifying the current limitations: items empty, category defaults, local-first persistence.
