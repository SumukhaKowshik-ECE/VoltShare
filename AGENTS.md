# VoltShare App Guidelines

## Stack
- Frontend: Flutter (iOS/Android)
- Backend: Firebase (Auth, Cloud Firestore, Cloud Functions)
- Location Services: Geoflutterfire / Firestore Geoqueries

## Coding Standards
- Use clean architecture in Flutter (`lib/features`, `lib/core`, `lib/services`).
- Prefer strongly typed data models with JSON serialization for Firestore documents.
- Keep UI components separate from business logic (use Provider, Riverpod, or Bloc).