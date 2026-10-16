# Instructions for coding agents

## Purpose and current state

Build a functional Photo-to-Minifigure MVP: photographs become personalized, original brick-toy-inspired characters, synchronized 2D outlines, and validated 3D-printable models. Face Only produces a head; Full Body produces a complete stylized character. Use an original aesthetic and original assets without official LEGO branding or proprietary geometry.

This repository currently contains planning documents, configuration, and empty directory placeholders only. No phase of application implementation has begun. Do not infer functionality from a directory, an API proposal, a sample parameter payload, or a future verification command. Implement only when the user authorizes implementation; initialization alone does not authorize Phase 1.

## Mandatory working rules

1. Inspect existing files, applicable agent instructions, relevant tests, and repository status before modifying code. Preserve unrelated user work.
2. Read [PLANS.md](PLANS.md) before implementing significant features. Consult the relevant product, architecture, and printing documents.
3. Keep PLANS.md updated as implementation progresses. Mark a task complete only with evidence that its acceptance criteria and relevant checks pass; record blockers and scope changes honestly.
4. Prioritize a working end-to-end slice over unnecessary complexity. First make one real path work, then generalize within the MVP.
5. Never fake image analysis, progress, confidence, or generation results. Test doubles belong in tests and must not become production fallbacks.
6. Never substitute a static example model for a personalized model without clearly indicating that it is an example. A failed analysis stays a visible failure; an explicitly chosen manual workflow must say it is manual.
7. Validate STL geometry before describing it as printable or enabling its download. A skipped, unknown, stale, or failed required check blocks export.
8. Keep the frontend preview consistent with exported geometry. Preview and STL must derive from the same final mesh revision; materials alone cannot represent required printable details.
9. Use modular, reusable code with explicit ownership of contracts and side effects.
10. Run relevant tests and checks after changes. Report what ran, the result, and anything that could not run. Do not claim unexecuted checks passed.
11. Avoid unnecessary dependencies, services, abstractions, and duplicated schema definitions. Add a dependency only with a concrete benefit and compatibility/license review.
12. Document important architectural decisions and consequential changes in [docs/DECISIONS.md](docs/DECISIONS.md).
13. Keep work inside the defined MVP. Explain and record any necessary scope exception; do not silently add product features.
14. Do not permanently store uploaded personal photos by default. Delete input/intermediate images after analysis on success, error, cancellation, or timeout; implement crash cleanup.
15. Do not expose photographs to external AI services without explicit user approval. Initial inference stays local to the backend and requires no paid API.

## Product scope

Required: both generation modes; real local landmark/appearance analysis; editable structured parameters; outlines; procedural geometry; interactive 3D viewing; basic face, hair, clothing, and color controls; validated STL and GLB downloads; SVG/PNG outline downloads; responsive, understandable workflows.

Excluded: authentication, user accounts, payments, shipping, orders, ecommerce, permanent photo libraries, unrestricted AI meshes, articulated assemblies, exact likeness claims, and proprietary connector compatibility. A local anonymous session and ephemeral job capabilities are sufficient. Prefer a single fused display model for the MVP.

## Stack and architecture

- Frontend: React, TypeScript, Vite, Tailwind CSS; Three.js through React Three Fiber and Drei.
- Backend: Python, FastAPI, Pydantic, OpenCV, Pillow, MediaPipe, NumPy, Trimesh, Shapely; Manifold3D where solid Boolean operations are needed.
- Planned verification: ESLint/Prettier, TypeScript, Vitest/Testing Library, Playwright, Ruff, mypy, pytest.
- Confirm and pin compatible runtime/library versions in Phase 1. Do not assume the newest versions form a compatible CV/geometry stack.
- Pipeline: photo upload -> image analysis -> structured character parameters -> shared 2D outlines/profiles -> procedural 3D geometry -> validation -> STL export.
- Canonical versioned parameters belong to backend schemas. Generate frontend API types from OpenAPI when implemented. Validate all requests on the backend regardless of browser validation.
- Keep HTTP routes thin. Pure analysis/mapping/profile/geometry functions must not depend on HTTP requests, browser state, or storage implementations.
- The server creates final geometry once; outline, preview, validation report, and downloads identify the same immutable revision. The browser renders server GLB rather than implementing a second final mesh generator.
- Keep CPU work outside the API event loop. Use a bounded local job worker and temporary files; do not introduce Redis, Celery, a database, or microservices without evidence.

## Repository ownership and component organization

| Location | Responsibility |
| --- | --- |
| `frontend/src/app/` | Application composition and small workflow state |
| `frontend/src/components/ui/` | Reusable accessible presentation primitives |
| `frontend/src/features/upload/` | Mode, file validation feedback, upload flow |
| `frontend/src/features/character/` | Parameter review, outlines, customization |
| `frontend/src/features/viewer/` | R3F scene, GLB lifecycle, print inspection |
| `frontend/src/features/exports/` | Validation status, artifact selection/download |
| `frontend/src/lib/api/` | Typed HTTP client, error decoding, job polling |
| `frontend/src/types/` | Generated API contracts and frontend-only types |
| `frontend/src/test/`, `frontend/tests/e2e/` | Test setup and browser tests |
| `backend/app/api/` | Versioned HTTP endpoints and dependency wiring |
| `backend/app/core/` | Configuration, application lifecycle, errors, privacy |
| `backend/app/schemas/` | Authoritative request/result/parameter schemas |
| `backend/app/services/analysis/` | Decode, local landmarks, extraction, mapping |
| `backend/app/services/outlines/` | Shared profiles, face/body outline output |
| `backend/app/services/geometry/` | Bounded procedural solids and assembly |
| `backend/app/services/validation/` | Topology, dimensions, manufacturing checks |
| `backend/app/services/exports/` | Artifact serialization and manifests |
| `backend/app/services/jobs/` | Worker lifecycle, bounded resources, temporary artifacts |
| `backend/models/` | Ignored downloaded model assets; document provenance before use |
| `backend/tests/unit/`, `backend/tests/integration/`, `backend/tests/fixtures/` | Pure tests, pipeline/API tests, approved fixtures |
| `docs/`, `scripts/` | Living documentation and task-specific tooling |

Colocate feature-specific components, hooks, and tests with their feature. Promote code to shared locations only when there is an actual shared responsibility. Do not create generic repositories, event buses, or service base classes speculatively.

## Frontend and TypeScript conventions

- Enable strict TypeScript. Use explicit domain types, discriminated unions for modes/job states, and `unknown` plus validation at untrusted boundaries. Avoid `any`, unchecked casts, non-null assertions, and suppression comments without justification.
- Use function components and hooks. Component/type names use PascalCase; functions/variables/hooks use camelCase and `use...` for hooks. Use consistent descriptive file names; React component files use PascalCase and `.tsx`.
- Use named exports for internal components/helpers; permit framework-required default exports. Keep imports acyclic and use type-only imports where appropriate.
- Use local state, reducers, and focused context before adding a state library. Separate draft controls from the last server-confirmed revision.
- Keep loading, empty, error, expired, and validation-blocked states explicit. Cancel polling and object URLs on teardown; discard stale asynchronous results after a newer edit.
- Use Tailwind with a small original design vocabulary. Preserve labels, keyboard operation, focus visibility, sufficient contrast, and text equivalents for canvas information.
- Render GLB using R3F/Drei; dispose replaced GPU resources. Orbit/zoom and visual helpers must not modify printable geometry. Show units and expose a print view without decorative materials.
- Do not log photos, job capability URLs, feature vectors, or full parameter payloads to analytics or the console. Do not put secrets in `VITE_*` variables.
- Phase 1 should configure ESLint, Prettier, and repeatable scripts. Follow `.editorconfig`: UTF-8, LF, two spaces for TS/JS/JSON/CSS, final newline; use formatter output rather than manual style churn.

## Backend and Python conventions

- Use type annotations for public functions and service boundaries, Pydantic schemas for external data, and small pure functions for numeric work. Avoid global mutable request state.
- Use snake_case for files/functions/variables, PascalCase for classes, and UPPER_SNAKE_CASE for constants. Use four spaces and a planned Ruff line length of 100.
- Prefer `pathlib`, context managers, explicit resource ownership, and `try/finally` for cleanup. Never trust a client filename as a filesystem path.
- Separate decoding, detection, extraction, mapping, profile construction, solids, validation, and serialization. Document numeric assumptions, coordinate systems, tolerances, and units.
- Use float64 for geometry computation where practical; explicitly account for STL/GLB serialization precision. Reject NaN/infinity and out-of-range dimensions. Use NumPy deliberately; avoid accidental mutation of caller arrays.
- Keep async routes for I/O coordination. Run CPU-bound CV and mesh operations in bounded workers with time/memory/resource limits; an async declaration alone does not make CPU work nonblocking.
- Resolve backend paths against the repository root, not the current working directory. Load only documented root environment settings and fail clearly on invalid configuration.
- Phase 1 should configure Ruff, mypy, pytest, packaging, and a reproducible dependency lock. Do not add these manifests solely to make initialization appear runnable.

## API conventions

Follow the proposed endpoint and envelope contracts in [ARCHITECTURE.md](docs/ARCHITECTURE.md). Use `/api/v1`, JSON `snake_case` properties, explicit `schema_version`, stable error codes, and appropriate HTTP status codes. Health is `/healthz`.

Use multipart for one photo plus mode; use JSON for generation parameters. Never accept remote image URLs, client-selected output paths, executable SVG, arbitrary mesh uploads, or arbitrary geometry scripts in the MVP. Validate file signatures, decoded pixel count, byte count, dimensions, and supported modes/styles.

Jobs return `202` and a high-entropy capability identifier; polling is honest about stages rather than fabricated percentages. Expiring capabilities are not accounts. No job listing or public artifact directory. Restrict CORS to explicit development origins, keep local binding as the default, and return private/no-store responses for sensitive results. Revisit deployment protections before exposing the API publicly.

## Geometry and manufacturing requirements

Treat [PRINTING.md](docs/PRINTING.md) as the manufacturing contract. All numeric targets there are provisional until verified; do not invent universal printer tolerances.

- Internal dimensions and STL coordinates use millimeters; GLB uses meters with an explicit axis/scale conversion. Never silently rescale exports or rely on STL metadata for units.
- Default to solid, fused, non-articulated parts: one connected shell, positive volume, consistent outward winding, no open boundaries, non-manifold vertices, zero-area faces, or self-intersections.
- Union overlapping solids using a checked solid Boolean engine. Concatenating intersecting watertight meshes is not a valid final union. Fail clearly if a required Boolean/validation capability is missing.
- Keep decorative features thick enough at the final scale. Required facial/clothing relief must exist in the mesh; color-only customization is clearly labeled as absent from STL.
- Validate final serialized STL after reloading it, including dimensions/topology and a profile-aware feature check. Attribute every check to a report with a revision hash. Revalidate any mesh repair and any change that affects geometry.
- Watertightness alone is insufficient. Unknown intersection or thickness results block the printable claim. Use supported template bounds plus a documented conservative validation strategy; do not assert that a heuristic proves arbitrary mesh printability.
- Include support/orientation advisories and original dimensions. Describe successful validation narrowly; actual slicer/physical verification must be reported separately.

## Image processing, security, and privacy

- Support one person per image; detect ambiguity rather than selecting a face silently. Face Only needs a usable face; Full Body needs usable face and body evidence. Provide actionable guidance for blur, occlusion, cropping, multiple people, or missing landmarks.
- Normalize EXIF orientation, strip metadata, enforce decode limits before large allocations, convert color spaces explicitly, and reject corrupt/unsupported/animated inputs. Do not trust MIME or extensions alone.
- Use local MediaPipe task models with recorded version, origin, checksum, and license. Download models during documented setup, never by transmitting user images. A missing model is a configuration error, not a reason to fabricate outputs.
- Map observable ratios, landmarks, and sampled colors into a bounded original style vocabulary. Hair/clothing suggestions may use limited heuristics; mark their uncertainty and require review where needed.
- Track per-field source (`observed`, `heuristic`, `user`, `default`) separately from parameters. Do not invent calibrated confidence scores or infer identity, ethnicity, health, gender, or real body measurements.
- Delete raw and normalized images after analysis; retain only necessary parameters and derived artifacts temporarily. Do not commit personal photos, landmarks, logs, models, temporary output, or generated exports.
- Test cleanup for success, failure, timeout, deletion, and startup recovery. Use fixed maximum lifetimes and disk/queue limits. Derived meshes and outlines can also be identifying; treat them as private data.

## Error handling and observability

Expose stable machine-readable error codes, a safe message, retryability, and an opaque request identifier. Include field errors or a validation report when safe. Never return stack traces, local paths, image bytes, or raw third-party exceptions to the browser.

Distinguish invalid input, no/multiple subjects, insufficient landmarks, model unavailability, generation failure, validation failure, busy queue, timeout, and expired artifacts. Log stage, duration, safe code, and resource summaries without personal payloads or capability tokens. A failed export must never yield a successful download link.

## Testing expectations

Write tests for behavior and risks, not to mirror implementation details. Every phase adds relevant tests as functionality lands; Phase 7 consolidates coverage and verification rather than postponing testing until the end.

- Frontend: mode flow, state transitions, parameter editing, stale-response handling, accessible controls, validation/export gating, and error/expiry recovery.
- Backend: strict schemas, limits, color/coordinate transforms, deterministic mapping/profiles/geometry, API contracts, and job cleanup.
- Geometry: both modes and supported option/size boundaries; true union, vertex/edge manifoldness, intersection checks, positive volume, minimum features, STL reload, GLB/STL agreement after explicit transforms.
- CV: consent-safe fixtures covering representative lighting/appearances; no face, multiple people, blur, occlusion, cropped body, corrupt bytes, and unsafe sizes. Assert observable output and honest warnings, not perfect likeness.
- End-to-end: actual local analysis through outline, GLB, and validated STL in both modes; at least one edit changes the corresponding personalized output. Mocks alone cannot satisfy MVP acceptance.

Use planned verification commands in README and PLANS until actual scripts exist, then keep them synchronized. Initialization requires documentation/link/structure/whitespace checks only; do not install packages or pretend application tests can run.

## Documentation and change reports

Update README for real setup changes, PRD for product-scope changes, ARCHITECTURE for contracts/data flow, PRINTING for manufacturing assumptions, DECISIONS for consequential choices, and PLANS for progress. Keep planned versus implemented behavior explicit.

For completed implementation work, report the behavior changed, relevant files, checks and their results, and material limitations. Add examples only when labeled as illustrative; never present a documented sample as a computed photo result.
