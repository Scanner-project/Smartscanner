# SmartScanner Architecture

## Pattern
This codebase uses MVVM with Provider.

## Layers and Responsibilities
- UI layer: [lib/main.dart](lib/main.dart), [lib/components](lib/components)
  - Renders state and forwards user actions.
  - Must not call databases, Firestore, ML Kit, or SharedPreferences directly.
- ViewModel layer: [lib/view_models/app_state.dart](lib/view_models/app_state.dart)
  - Owns UI state, coordinates use cases, notifies listeners.
  - Must not contain direct persistence or external API wiring.
- Service layer: [lib/services](lib/services)
  - Implements business logic and integrations.
  - Includes [lib/services/receipt_processing_service.dart](lib/services/receipt_processing_service.dart), [lib/services/receipt_repository.dart](lib/services/receipt_repository.dart), [lib/services/storage_service.dart](lib/services/storage_service.dart), [lib/services/gemini_service.dart](lib/services/gemini_service.dart).
- Domain models: [lib/types.dart](lib/types.dart)
  - Shared contracts between layers.

## Boundary Rules
1. UI -> ViewModel only
- Allowed: `context.watch<AppState>()`, `context.read<AppState>()`.
- Forbidden: importing Service files in UI widgets.

2. ViewModel -> Services only
- Allowed: orchestrating service calls, mapping results to UI state.
- Forbidden: direct SharedPreferences or Firestore operations.

3. Services -> Data/External systems
- Allowed: Firestore, SharedPreferences, ML Kit, external APIs.
- Forbidden: importing UI widgets or ViewModel classes.

4. Models are shared contracts
- Keep entities and value objects in [lib/types.dart](lib/types.dart).

## Data Flow
1. User action in UI.
2. UI calls ViewModel method.
3. ViewModel calls service methods.
4. Services read/write local or remote sources.
5. ViewModel updates state and notifies listeners.
6. UI rebuilds.

## Repository Layout
- [lib/main.dart](lib/main.dart)
- [lib/components](lib/components)
- [lib/view_models](lib/view_models)
- [lib/services](lib/services)
- [lib/types.dart](lib/types.dart)
- [docs/adr](docs/adr)

## Rules for New Work
- Any cross-layer change must preserve the direction UI -> ViewModel -> Services.
- Major architecture decisions must be recorded in an ADR under [docs/adr](docs/adr).
- If boundaries change, update this file and add a new ADR.
