# Integration Guide (optional)

This short guide covers wiring an external LLM/assistant (e.g., Gemini-style service) and recommendations for unifying local vs Firestore storage.

## Gemini / LLM integration (not currently present in code)
- Purpose: use an external LLM to improve receipt parsing (item extraction, categorization), prompt-based correction, or verification.
- Recommended approach:
  1. Feature-flag integration behind a runtime config or environment variable (e.g., `ENABLE_GEMINI=true`).
 2. Store API keys in secure platform mechanisms (Android: encrypted preferences / keystore; iOS: Keychain). Avoid hardcoding keys in the repo.
 3. Implement a thin service `lib/services/llm_service.dart` with a simple interface:
     - `Future<ParsedReceipt> parseReceiptWithLLM(String rawText)`
     - `Future<Category> categorizeItems(List<ReceiptItem>)`
 4. Use request batching and concurrency limits to avoid quota spikes.

## Storage read/write unification recommendation
- Current behavior: local-first (SharedPreferences) for app UX; receipts are appended to Firestore but not treated as canonical source.
- Option A (Local-first, sync-to-cloud): keep SharedPreferences as the UX-first store and keep publishing receipts to Firestore asynchronously. Add reconciliation that maps Firestore document ids back to local receipts.
- Option B (Cloud-first with offline cache): treat Firestore as canonical and migrate local storage to a cache layer (e.g., Hive or SQLite) that supports merging and sync. This increases complexity but improves multi-device consistency.

Recommendation: start with Option A and implement a small sync layer:
- When publishing a receipt, await the Firestore add result, capture the doc id, and store it into the local `Receipt` (field `id` or separate `firestoreId`).
- Add a background reconcile task that compares local receipts with Firestore collection and re-publishes missing items or updates metadata (linkedReceiptId, isProcessed).

## Checklist for PR
- Add `llm_service.dart` scaffold (feature-flagged).
- Add secure API key instructions to README and platform-specific setup.
- Implement Firestore doc id persistence mapping and a reconcile job.
