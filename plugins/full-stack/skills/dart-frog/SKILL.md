---
name: dart-frog
description: "Dart Frog API routes, middleware, request/response, and context DI. Prefer the dart_frog CLI from the API package root. Use when creating or editing routes, middleware, form data, file uploads, or when the user mentions dart_frog, API routes, or Dart backend."
---

## Related Guidelines

- `@data` — Data layer patterns for APIs and clients
- `@di` — Injectable/GetIt wiring
- `@dart` — Dart language conventions

# Dart Frog

Expert guidance for the Dart Frog backend framework: routes, middleware, request/response, and DI.

## CLI-first workflow

**Default to the `dart_frog` CLI** for anything it supports instead of improvising file trees or shell shortcuts. Run commands from the **Dart Frog package root** (where `routes/` and `pubspec.yaml` live for that API).

| Task | Command |
|------|---------|
| Local server | `dart_frog dev` |
| Production bundle | `dart_frog build` |
| Inspect registered routes | `dart_frog list` |
| New route | `dart_frog new route "/path"` |
| New middleware under a segment | `dart_frog new middleware "/path"` |
| New Dart Frog app (greenfield) | `dart_frog create <name>` |

Optional: `dart_frog daemon` for editor/tooling integrations that expect the daemon.

Use `dart_frog --help` and `dart_frog help <command>` when flags or behavior are unclear. Only hand-create or edit files when the CLI has no equivalent (e.g. implementing handler logic, custom middleware chains).

## Route structure

- **File-based routing**: `routes/` mirrors URLs. `routes/products/index.dart` → `GET/POST /products`; `routes/products/[id].dart` → `GET/PATCH/DELETE /products/:id`.
- **Handler signature**: `Future<Response> onRequest(RequestContext context)` or `Response onRequest(...)` for sync. For dynamic segments: `onRequest(RequestContext context, String id)`.
- **Method dispatch**: Use `switch (context.request.method)` and handle `HttpMethod.get`, `HttpMethod.post`, `HttpMethod.patch`, `HttpMethod.delete`; return `Response(statusCode: HttpStatus.methodNotAllowed)` for others.

## Scaffolding details (`dart_frog new`)

- **`dart_frog new route "/new-route"`** — creates the route folder and stub handler(s) under `routes/` for that path.
- **`dart_frog new middleware "/existing-segment"`** — adds or scaffolds `routes/<segment>/_middleware.dart` (e.g. `"/inventory-lots"` → `routes/inventory-lots/_middleware.dart`). Then edit the file to chain `provider<T>` and other middleware.

Prefer these over creating empty files manually so paths and stubs match Dart Frog conventions.

## Request handling

### JSON body
```dart
final body = await context.request.json();
if (body is! Map<String, dynamic>) {
  return Response(statusCode: HttpStatus.badRequest, body: 'Expected JSON object');
}
```

### Form data (multipart)
```dart
final formData = await context.request.formData();
final { 'field_name': String? value } = formData.fields;
// Optional file: formData.files['key'] → UploadedFile (readAsBytes(), contentType)
```

### File uploads
- `formData.files['key']` returns `UploadedFile?`. Use `uploadedFile.readAsBytes()` and `uploadedFile.contentType` (e.g. `ContentType(:mimeType)` for `mimeType` string).

## Dependency injection

- **Provide in middleware**: `handler.use(provider<MyRepo>((context) => sl<MyRepo>()));`
- **Read in route**: `final repo = context.read<MyRepository>();`
- Wire external packages (e.g. GetIt/sl) in `main.dart` or an `init` callback; use middleware only to expose instances to `context.read<T>()`.

## Response patterns

- **JSON**: `Response.json(body: object)` (object may be `Map` or model with `toJson()`). For status: `Response.json(body: ..., statusCode: HttpStatus.created)`.
- **Status only**: `Response(statusCode: HttpStatus.noContent)` or `Response(statusCode: HttpStatus.notFound)`.
- **Error body**: `Response(statusCode: HttpStatus.badRequest, body: message)`.
- Prefer returning JSON-serializable data (e.g. `product.toJson()` or `body: product` if the framework supports it) and consistent status codes (404 for not found, 400 for validation, 500 for server errors).

## Dynamic route parameters

- File `routes/products/[id].dart` → parameter `id` (String). Parse with `int.tryParse(id)`; return 400 if invalid before calling repository.
- Use the same `RequestContext context` and add positional parameters for each segment.

## Middleware

- **Per-route**: `routes/products/_middleware.dart` applies to `/products` and children. Use `Handler middleware(Handler handler) => handler.use(...).use(...);` For a new `_middleware.dart` under an existing folder, run `dart_frog new middleware "/products"` (see **CLI-first workflow** and **Scaffolding details**).
- **Built-in**: `requestLogger()` for request logging.
- **Provider**: `provider<T>((context) => instance)` so routes can `context.read<T>()`.

## Conventions

- Use `dart:io` for `HttpStatus` when needed.
- Keep route handlers thin: parse request, call repository or service, return response.
- Catch domain exceptions (e.g. `ProductNotFoundException`) and map to 404; avoid leaking stack traces to clients.
- For multipart create/update, validate required fields and return 400 with a clear message when invalid.

## Build and deploy

- Use the CLI: **`dart_frog dev`** locally, **`dart_frog build`** for production (output in `build/`). Deploy that build output per your hosting (e.g. container from `build/`).

## Checklist for new routes

- [ ] `dart_frog new route "/your-path"` (avoid hand-creating `routes/` layout unless necessary)
- [ ] Handle allowed methods and return 405 for others
- [ ] Validate path/query/body and return 400 when invalid
- [ ] Use `context.read<T>()` for repos; provide in `_middleware.dart` if needed
- [ ] Return JSON or appropriate body and status code (200, 201, 204, 400, 404, 500)
