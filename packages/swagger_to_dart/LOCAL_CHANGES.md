# Local changes (fork notes)

This is a fork of the original `swagger_to_dart` package. The changes below were applied locally and are not upstream.

---

## Fix: extension wrapper methods omit query parameters in forwarding call

**File:** `lib/src/generator/api_client/api_client_generator.dart`  
**Around line:** ~288

### Problem

When an endpoint uses `multipart/form-data`, the generator emits two things:

1. A real `@RestApi` method named `methodName_` (with trailing underscore) that carries the `@Part`, `@Query`, etc. annotations.
2. A convenience wrapper method named `methodName` (no underscore) inside an extension, which accepts the typed request body and forwards the call to `methodName_`.

The wrapper's forwarding call was hardcoded to only pass `requestBody`, `extras`, optionally `queries`, and the progress/cancel callbacks. **All other query parameters (e.g. `providerId`, `dryRun`, `skipped`) were silently dropped**, so they were never sent to the server.

### Fix

Changed the `Code(...)` body of the generated extension method to iterate over all `parameters` that are not the special `queries` parameter and emit `paramName: paramName,` for each one, so every query param declared in the OpenAPI spec is forwarded correctly.

```dart
// Before
'''return ${methodName}_(..., extras: extras,
  ${hasQueries ? 'queries: queries,' : ''}
  cancelToken: cancelToken, ...);'''

// After
'''return ${methodName}_(..., extras: extras,
  ${parameters.where((e) => e.name != _queriesParameterName).map((e) => '${e.name}: ${e.name},').join('\n')}
  ${hasQueries ? 'queries: queries,' : ''}
  cancelToken: cancelToken, ...);'''
```
