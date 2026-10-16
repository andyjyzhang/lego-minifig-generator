# Implementation plan

Status: **all implementation phases are unstarted**. This initialization provides a plan, documentation, and placeholders only. Every implementation and acceptance checkbox remains unchecked. Commands below are future interfaces, not currently executable project checks.

Read [AGENTS.md](AGENTS.md), [PRD](docs/PRD.md), [ARCHITECTURE](docs/ARCHITECTURE.md), [PRINTING](docs/PRINTING.md), and [DECISIONS](docs/DECISIONS.md) before significant implementation. Update this plan with evidence as work lands; an implemented task is not complete until its relevant acceptance checks pass.

## Sequence and dependency rules

`Phase 1 -> Phase 2 -> Phase 3 -> Phase 4 -> Phase 5 -> Phase 6 -> Phase 7`

This is the delivery order. Testing and essential error/privacy controls accompany every phase. Phase 6 refines an already usable flow; Phase 7 consolidates verification. Phase 2 uses explicitly labeled manual/synthetic parameter inputs to prove geometry; photo-driven personalization begins in Phase 3. Do not represent Phase 2 as photo generation.

Phase 2 introduces internal 2D profiles before solid construction. Phase 4 publishes those same profiles as polished outline artifacts rather than inserting a separate, inconsistent outline generator. Phase 5 uses the same revisioned generation pipeline rather than frontend-only mesh edits.

All commands run from the repository root with the future virtual environment activated. Phase 1 must create the listed npm scripts, Python tool configuration, and test markers before those commands become acceptance checks. Introduce named test files alongside their phase; a command that collects zero tests does not pass acceptance. Revisit provisional limits and record decisions with measurements rather than silently relaxing validation.

## Phase 1 — Project Foundation

**Objective:** establish a reproducible frontend/backend development loop and a real API connectivity slice.

**Dependencies:** initialization documents; runtime/package compatibility decision D-009; no preceding implementation.

Tasks:

- [ ] Confirm supported Node/Python versions and Windows/Linux compatibility for MediaPipe, NumPy, Trimesh, Shapely, and Manifold3D; record evidence and pin versions.
- [ ] Initialize React/TypeScript/Vite, strict TypeScript, Tailwind, and the planned viewer dependencies without implementing later product features.
- [ ] Initialize the installable FastAPI package, settings validation, lifespan management, and `/healthz`.
- [ ] Add reproducible npm/Python dependency manifests and lockfiles; configure lint, format, typecheck, unit-test, and build commands.
- [ ] Implement root `.env` loading, explicit relative-path resolution, Vite `/api` and `/healthz` proxies, explicit CORS origins, local binding, and safe configuration errors.
- [ ] Add shared accessible button, form field, status, and error primitives; create the application shell.
- [ ] Add a typed connectivity request with visible loading/error/retry states and request identifiers.
- [ ] Add frontend connectivity tests, `backend/tests/integration/test_health.py`, and safe startup/settings tests.
- [ ] Update README with verified setup instructions and actual commands; keep later features labeled planned.

Acceptance criteria:

- [ ] A clean checkout can start both development servers using documented commands on the declared platforms.
- [ ] The browser reaches the real backend and recovers visibly from an unavailable backend; health exposes no private configuration.
- [ ] Formatting, lint, types, tests, and production frontend build pass; no phase relies on paid services.

Planned verification:

```powershell
npm --prefix frontend run lint
npm --prefix frontend run format:check
npm --prefix frontend run typecheck
npm --prefix frontend run test -- --run
npm --prefix frontend run build
python -m ruff check backend
python -m ruff format --check backend
python -m mypy --config-file backend/pyproject.toml backend/app
python -m pytest backend/tests/integration/test_health.py
```

Also start both servers using README and inspect `/healthz` and browser connectivity.

## Phase 2 — Procedural 3D Generation

**Objective:** produce deterministic, validated head and full-body solids from explicit parameters before adding photo inference.

**Dependencies:** Phase 1; schema and print-profile contracts; resolve D-004/D-005 validation strategy before claiming printable output.

Tasks:

- [ ] Implement versioned strict Pydantic character parameters, mode-specific rules, numeric ranges, supported styles, and frontend type generation from OpenAPI.
- [ ] Add immutable parameter revision hashing including generator/profile versions and client revision echoing.
- [ ] Implement shared 2D head/face/body profiles with documented coordinates and invalid-profile rejection.
- [ ] Generate head, face relief, and simplified hair geometry with original proportions.
- [ ] Generate a neutral full body with fused head, torso, arms/hands, legs/feet, and structurally sound connections.
- [ ] Implement checked solid unions, bounded tessellation, final-scale feature constraints, and failure when required geometry capabilities are unavailable.
- [ ] Implement bounded ephemeral generation jobs, capability IDs, stage polling, queue limits, timeouts, private artifact delivery, deletion, and cleanup.
- [ ] Generate GLB and binary STL from the same final mesh; apply documented unit/axis transforms and reload/validate STL before publishing it.
- [ ] Produce revisioned validation reports covering topology, intersections, dimensions, connected components, and profile-aware minimum features.
- [ ] Add R3F/Drei GLB viewing with orbit/zoom, dimensions, validation status, geometry inspection, and stale-result protection.
- [ ] Expose manual parameter input only as an explicitly labeled development/manual workflow; never imply photo analysis occurred.
- [ ] Add `test_parameters.py`, `test_geometry.py`, `test_exports.py`, and `test_generation_jobs.py` covering both modes and invalid cases; mark numeric geometry/export tests `geometry`.

Acceptance criteria:

- [ ] Both modes generate one connected, positive-volume solid at documented dimensions for supported parameter boundaries.
- [ ] Preview and STL agree after explicit unit/axis conversion; the validation report identifies the exact exported bytes and revision.
- [ ] Open, self-intersecting, non-manifold, disconnected, undersized, unknown-check, or stale outputs cannot be downloaded as validated STL.
- [ ] No intersecting-mesh concatenation is accepted as a union; cleanup and timeout tests pass.

Planned verification:

```powershell
python -m pytest backend/tests/unit/test_parameters.py backend/tests/unit/test_geometry.py backend/tests/unit/test_exports.py backend/tests/integration/test_generation_jobs.py
npm --prefix frontend run typecheck
npm --prefix frontend run test -- --run
npm --prefix frontend run build
```

Inspect one head and one body in the viewer and a slicer; record measured bounding boxes and support advisories. Do not treat a slicer's automatic repair as validator success.

## Phase 3 — Computer Vision

**Objective:** replace manual inputs with honest, local photo-derived character suggestions in both modes.

**Dependencies:** Phases 1–2; approved local model assets and licenses/checksums; consent-safe fixture strategy D-007.

Tasks:

- [ ] Add single-photo JPEG/PNG upload and backend signature, byte, pixel, dimension, decode, and animation validation.
- [ ] Normalize orientation/color, strip metadata, bound analysis resolution, and remove all raw/intermediate images after analysis.
- [ ] Document local task-model setup, asset provenance and checksums; return configuration errors when models are missing.
- [ ] Implement face landmark detection with explicit missing/ambiguous-face outcomes.
- [ ] Implement body/pose landmark detection and usable-face/body requirements for Full Body; detect cropped or obscured inputs.
- [ ] Extract bounded face ratios, visible body proportions, and representative colors with explicit color-space transforms.
- [ ] Prototype limited hairstyle/clothing heuristics; report unsupported/uncertain values as defaults or review-needed suggestions rather than observations.
- [ ] Map results into canonical parameters with per-field provenance and honest confidence semantics; never infer sensitive identity attributes or metric body dimensions.
- [ ] Add analysis jobs to the existing worker lifecycle, bounded limits, progress stages, error envelope, and expiry/deletion behavior.
- [ ] Connect a real upload -> analysis -> review -> generation path; preserve user edits when appropriate and show resubmission guidance on failure.
- [ ] Add `test_upload_security.py`, `test_analysis.py`, and `test_analysis_jobs.py`; cover no/multiple people, corrupt/oversized images, occlusion, missing models, and cleanup.
- [ ] Verify offline inference after setup with representative approved photographs; document model limitations and low-confidence behavior.

Acceptance criteria:

- [ ] Each mode produces parameters derived from an actual accepted image and a visibly personalized result; labeled defaults never masquerade as detections.
- [ ] Full Body considers face, hair, outfit colors/styles, and visible body evidence, with user review wherever automatic inference is unsupported.
- [ ] Failed or ambiguous analysis yields actionable errors, with no silently substituted character or static model.
- [ ] Tests prove photos and intermediates are removed on success/failure/timeout/deletion and stale files are purged on startup.
- [ ] After model setup, no photo or analysis payload leaves the configured backend for an external service.

Planned verification:

```powershell
python -m pytest backend/tests/unit/test_analysis.py backend/tests/integration/test_upload_security.py backend/tests/integration/test_analysis_jobs.py
npm --prefix frontend run test -- --run
npm --prefix frontend run typecheck
```

Manually run both real photo flows offline after asset setup, inspect suggestion provenance, and verify cleanup in the configured temporary directory without adding private files to Git.

## Phase 4 — 2D Outline Generation

**Objective:** expose consistent face/full-body outlines as review artifacts and downloads.

**Dependencies:** shared profiles from Phase 2; real parameter mapping from Phase 3.

Tasks:

- [ ] Render face/head silhouette and facial curves from canonical profiles and parameters.
- [ ] Render full-body silhouette, hair, and simplified clothing boundaries from the same profile vocabulary used for solids.
- [ ] Define SVG viewBox, normalization, feature coordinates, safe escaping, and front-view orientation.
- [ ] Export safe self-contained SVG and bounded-resolution PNG; use existing Pillow drawing where practical before adding a rasterizer.
- [ ] Include outline artifacts in the existing generation manifest with the same revision and expiry.
- [ ] Show outlines in the review flow and invalidate them after relevant parameter changes.
- [ ] Add `test_outlines.py` covering both modes, malformed parameters, bounds, face/body alignment, SVG safety, and PNG dimensions.

Acceptance criteria:

- [ ] Both modes provide downloadable SVG and PNG generated from the current parameters; no raster tracing or static silhouette replaces personalization.
- [ ] Outline details and 3D features share the same coordinate definitions and parameter revision; outline regeneration does not rerun photo analysis.
- [ ] SVG contains no scripts, foreign content, external resources, or unescaped user data; PNG respects resource limits.

Planned verification:

```powershell
python -m pytest backend/tests/unit/test_outlines.py backend/tests/unit/test_geometry.py backend/tests/unit/test_exports.py
npm --prefix frontend run test -- --run
```

Compare the outline to the matching front-view mesh using documented projection/profile semantics; reopen both downloaded formats.

## Phase 5 — Character Customization

**Objective:** let users correct suggestions and personalize appearance without breaking geometry/export consistency.

**Dependencies:** Phases 2–4; a bounded, documented option catalog and validated print profiles.

Tasks:

- [ ] Add face controls for supported eyes, brows, mouth, and face-relief options within tested bounds.
- [ ] Add a small original hairstyle catalog and supported hairstyle controls, with explicit review for heuristic suggestions.
- [ ] Add simplified clothing options and skin/hair/eye/clothing color controls; label color-only effects as absent from STL.
- [ ] Add target size and print-profile controls constrained by final-scale minimum features.
- [ ] Separate draft from confirmed parameters; debounce regeneration, coalesce edits, and ignore older job responses.
- [ ] Refresh outline, preview, validation report, and export availability together; keep old results visibly stale until a new revision finishes.
- [ ] Add reset-to-analysis-suggestions behavior without retaining the original photo.
- [ ] Add boundary/combination tests for supported options and frontend tests for edits, errors, keyboard interaction, and race conditions.

Acceptance criteria:

- [ ] Face Only and Full Body allow supported edits and update visible results without another photo upload.
- [ ] A meaningful geometry edit changes the mesh; a color edit updates GLB/outline colors without falsely promising STL color.
- [ ] Unsupported combinations or insufficient thickness are rejected or explicitly adjusted with user-visible explanation; exports never use an obsolete report.

Planned verification:

```powershell
python -m pytest backend/tests/unit/test_parameters.py backend/tests/unit/test_geometry.py backend/tests/unit/test_outlines.py backend/tests/unit/test_exports.py
npm --prefix frontend run test -- --run
npm --prefix frontend run typecheck
npm --prefix frontend run build
```

Exercise rapid successive edits while a previous generation is running, then compare downloaded artifacts with the final displayed revision.

## Phase 6 — UX and Polish

**Objective:** make the complete flow understandable, responsive, and accessible.

**Dependencies:** Phases 1–5; measured stage timings and implemented failure paths.

Tasks:

- [ ] Add an original landing page describing both modes, approximate likeness, privacy, and printing limitations.
- [ ] Refine mode-specific photo guidance, upload review, suggestion confirmation, and restart/delete controls.
- [ ] Add honest queued/analysis/generation/validation stages and busy/timeout/retry states; no fabricated progress percentages.
- [ ] Improve recoverable input, failed-validation, backend-unavailable, and expired-artifact messaging with clear next steps.
- [ ] Test mobile/tablet/desktop layouts and provide usable fallback information when WebGL is unavailable.
- [ ] Complete keyboard/focus, contrast, labels, live-region, reduced-motion, and canvas text-equivalent checks.
- [ ] Dispose image URLs/GPU resources, bound polling, and measure reference-device generation/viewer performance.
- [ ] Document measured performance, limitations, privacy lifecycle, and support/orientation guidance.

Acceptance criteria:

- [ ] A first-time user can complete either flow with visible recovery from common failures on supported viewport sizes.
- [ ] Controls are keyboard operable and status/dimensions/validation are available outside the canvas.
- [ ] Results clearly distinguish geometric validation, print advisories, visual-only colors, uncertain analysis, and physical print evidence.

Planned verification:

```powershell
npm --prefix frontend run lint
npm --prefix frontend run test -- --run
npm --prefix frontend run build
python -m pytest backend/tests/integration
```

Record manual keyboard/responsive/WebGL fallback checks and measured latency against the PRD reference setup.

## Phase 7 — Testing and Verification

**Objective:** verify the complete MVP with real local inference and inspectable manufacturing evidence.

**Dependencies:** Phases 1–6; approved test images/models; reference hardware; slicer and physical printer access for the print trial.

Tasks:

- [ ] Review frontend behavioral coverage for both modes, uploads, customization, errors, stale jobs, expiry, and export gating.
- [ ] Review backend unit/integration coverage for contracts, CV, outlines, resource limits, deletion, worker shutdown, and cleanup after crashes.
- [ ] Run geometry matrices across supported styles, sizes, and profiles; include intentionally invalid and unknown-check cases.
- [ ] Add Playwright smoke tests using the real backend and approved images, without mocked analysis/mesh results, for both complete journeys.
- [ ] Verify GLB/STL coordinate, bounds, and geometry equivalence and exact-artifact validation reports.
- [ ] Slice exported head and body STLs at intended millimeter dimensions with no automatic geometry repairs; record slicer settings and observations.
- [ ] Physically print at least one representative model in the declared primary printer profile; record dimensions, feature survival, supports, and failures. If no printer is available, keep this criterion open and label print readiness provisional.
- [ ] Run the privacy/offline audit and inspect logs/Git history candidates for accidental personal artifacts.
- [ ] Add CI using the verified commands and model-fixture setup, with no paid service dependency or private photos.
- [ ] Reconcile README, PRD, ARCHITECTURE, PRINTING, DECISIONS, environment settings, actual schema/API, and this plan.

Acceptance criteria:

- [ ] All automated checks collect the intended tests and pass; both real end-to-end flows produce personalized outlines, preview, and validated STL.
- [ ] Invalid geometry is blocked server-side and in the UI; no success depends on a static model or silent slicer repair.
- [ ] Slicer and physical-print evidence supports the declared primary profile; other profiles remain explicitly provisional until verified.
- [ ] No private uploads/intermediates survive the documented lifecycle, and setup/inference work without paid AI APIs.

Planned verification:

```powershell
npm --prefix frontend run lint
npm --prefix frontend run format:check
npm --prefix frontend run typecheck
npm --prefix frontend run test -- --run
npm --prefix frontend run build
npm --prefix frontend run test:e2e
python -m ruff check backend
python -m ruff format --check backend
python -m mypy --config-file backend/pyproject.toml backend/app
python -m pytest backend/tests
python -m pytest backend/tests -m geometry
git diff --check
```

The geometry marker supports focused investigation; do not rerun it redundantly after the full suite unless changes or findings justify it. Preserve slicer/print evidence as documented, non-personal reports under `docs/` when available.

## MVP definition of done

- [ ] One usable face photo and one usable full-body photo each complete the actual local pipeline through parameters, outline, procedural mesh, validation, and downloads.
- [ ] The final results retain recognizable, image-derived stylized characteristics; uncertainty and user-selected/default values remain visible.
- [ ] A user can customize face/hair/clothing/colors within the supported catalog; current SVG, PNG, GLB, and STL outputs follow the same confirmed revision.
- [ ] STL downloads pass all required final-artifact checks for the chosen profile, show millimeter dimensions, and correspond to the preview geometry.
- [ ] Both modes have slicer evidence and the primary profile has a documented representative physical print; limitations are explicit.
- [ ] Upload validation, bounded resource use, private capability access, deletion/expiry, and all cleanup paths are verified.
- [ ] Responsive/accessibility checks, meaningful automated tests, real end-to-end smoke tests, and documentation review pass.
- [ ] Installation and inference require no paid AI API; no out-of-scope account or commerce features have been introduced.
- [ ] Every unresolved issue that affects the shipped support claims is resolved or narrows those claims explicitly in the documentation.
