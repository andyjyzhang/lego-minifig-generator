# Architecture decision log

Created 2026-10-15. `Accepted` means the design direction is chosen, not implemented. `Provisional` means the approach/values need measured validation. `Open` items have no settled implementation. Update an entry or add a superseding entry when evidence changes a decision; record consequences in PLANS and relevant documentation.

## D-001 — Modular frontend and backend

**Status:** Accepted for MVP.

Use a React/TypeScript/Vite/Tailwind frontend with Three.js, React Three Fiber, and Drei, paired with a modular Python/FastAPI backend. Keep API adapters, contracts, CV, outlines, geometry, validation, exports, and local jobs separate within one backend package.

**Why:** browser interaction and Python's CV/mesh tooling have distinct strengths; one backend process coordinator is sufficient for a local MVP.

**Alternatives:** browser-only CV/CAD complicates runtime/memory and export validation; microservices increase operational work before there is evidence of a need.

**Consequences:** API contracts and coordinate transforms require explicit tests; both development servers need documented startup and connectivity.

## D-002 — Canonical parameters and shared profiles

**Status:** Accepted for MVP.

Use versioned Pydantic character parameters as the source of truth and generate frontend types from OpenAPI. Shared 2D profile/curve definitions feed outline drawing and bounded solid construction. Use immutable revision IDs based on canonical parameters, generator, and profile versions.

**Why:** customization, outlines, validation, and exports must describe the same character. Sharing profiles avoids separate interpretations of facial/clothing features.

**Alternatives:** independent frontend mesh generation risks divergence; tracing a photo into arbitrary geometry provides poor manufacturing constraints.

**Consequences:** Phase 2 needs internal profiles even though polished outline artifacts arrive in Phase 4. Unsupported parameter values must be rejected, not silently ignored.

## D-003 — Local CV with explicit uncertainty

**Status:** Accepted for MVP.

Use Pillow/OpenCV/NumPy and local MediaPipe task models. Map landmarks and simple appearance heuristics into a limited original vocabulary. Store per-field provenance outside canonical parameters; use confidence only where the detector/method actually supplies a meaningful quantity. Request user review for uncertain hair/outfit/occluded features.

**Why:** usable personalization without paid API dependency or external photo transfer. [MediaPipe Face Landmarker requires local model assets and returns landmarks](https://developers.google.com/edge/mediapipe/solutions/vision/face_landmarker/python); it does not supply a complete personalized printable character.

**Alternatives:** unrestricted remote image/mesh generation adds privacy/cost uncertainty and weak geometry guarantees; manual-only creation does not satisfy photo-to-character requirements.

**Consequences:** asset licenses/checksums and setup are required. A single image cannot establish real dimensions or hidden geometry. No sensitive-attribute or identity inference.

## D-004 — Procedural fused solids and checked unions

**Status:** Accepted direction; engine integration requires Phase 2 verification.

Use Shapely for supported planar profiles and Trimesh for mesh inspection/serialization. Prefer Manifold3D for required solid Boolean operations; verify installation, numeric behavior, status handling, and provenance. Start with original, solid, non-articulated head/full-body display models.

**Why:** constrained templates support measurable dimensions and minimum features; true union removes overlapping internal surfaces. [Manifold's project documentation describes its solid geometry and Boolean operations](https://github.com/elalish/manifold).

**Alternatives:** concatenated overlapping meshes are not an acceptable union; unrestricted generated meshes are difficult to constrain; articulated/split models add tolerance and assembly work beyond MVP.

**Consequences:** a missing/failed engine blocks affected generation. Initial dimensions do not imply commercial toy compatibility. No placeholder result is substituted for a failed union.

## D-005 — Validation gate and printing targets

**Status:** Accepted gate; provisional numerical profiles; exact robust checks Open.

Every STL requires final-byte reload validation and a revisioned report. Watertightness alone is insufficient. Required unknown checks block STL publication. Use initial FDM/resin dimensions and feature targets in [PRINTING.md](PRINTING.md), with FDM proposed as primary. Propose 24 mm heads/80 mm bodies, including hair, and bounded size ranges.

**Why:** export claims need evidence about actual surfaces and final scale, not only an attractive preview.

**Alternatives:** slicer repair as the validator hides generator faults; declaring all closed meshes printable overstates support.

**Consequences:** Phase 2 must settle and verify robust self-intersection and feature-thickness coverage, plus numerical tolerances and final-file comparisons. Phase 7 must document slicer and primary-profile physical print evidence. Resin support remains provisional until trialed.

## D-006 — Server-owned preview/export mesh

**Status:** Accepted for MVP.

Generate a final mesh on the backend, then derive GLB and STL from that surface. Use millimeters/Z-up internally and in STL, with explicit meters/Y-up conversion for GLB. Permit material-only GLB coloration without implying STL color.

**Why:** one geometry implementation makes export parity testable. [glTF specifies meters and its coordinate convention](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units).

**Alternatives:** frontend procedural preview plus a separate backend export duplicates geometry logic; STL as the sole preview asset loses material presentation.

**Consequences:** regeneration is asynchronous/debounced, with draft/current revision handling. Materials may partition the final surface but must not add/remove geometry.

## D-007 — Temporary anonymous jobs and privacy

**Status:** Accepted lifecycle; limits provisional.

Use high-entropy job capabilities, private temporary artifacts, immediate photo cleanup after analysis, one-hour fixed result/artifact expiry, explicit deletion, and startup purge of prior-run temporary data. No user/account database or permanent photo store. Use only synthetic or explicitly licensed/consented fixtures; record fixture provenance before committing images.

**Why:** minimizes handling of identifying input while supporting polling and downloads.

**Alternatives:** persistent galleries and account sessions add data/operational scope; a purely synchronous HTTP request handles CPU jobs and timeouts poorly.

**Consequences:** server restart loses jobs/results; refresh can lose unsaved client edits; capabilities must not be logged/shared; no public listing. Derived models also merit privacy controls. Deletion is normal filesystem/application cleanup, not a forensic secure-erasure claim.

## D-008 — Bounded local CPU worker

**Status:** Accepted direction; initial limits provisional.

Keep one API coordinator with an in-memory bounded queue and a supervised local worker subprocess for each active CPU job, initially one active and four queued. Enforce a queue-plus-run deadline and terminate overdue work; workers own no durable state. Avoid Redis/Celery for the MVP.

**Why:** prevent blocking the API while preserving explicit timeout/cleanup control. FastAPI's documentation distinguishes heavy computation from small [background tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/#caveat).

**Alternatives:** CPU work inside async routes blocks service; a simple future timeout without stopping the process leaves resources in use; a distributed queue is unnecessary for a controlled local demo.

**Consequences:** run one coordinator, not multiple independent API workers. Verify Windows process spawning, deletion races, model-loading overhead, and memory/disk limits. Benchmark before introducing a persistent worker pool.

## D-009 — Reproducible setup and tooling

**Status:** Accepted tooling direction; exact versions and Python lock tool Open.

Use strict TypeScript/ESLint/Prettier and Python typing/Ruff/mypy/pytest, plus Vitest/Testing Library and Playwright for meaningful frontend coverage. Use npm with a committed lockfile; make the backend installable through `backend/pyproject.toml` and commit a reproducible Python lock. Select exact versions only after a Windows/Linux compatibility check, initially investigating Python 3.11 and a supported Node LTS release.

**Alternatives:** blindly selecting latest versions risks incompatible CV/geometry wheels; adding manifests/tool installations during planning violates initialization scope.

**Consequences:** Phase 1 selects pip-tools or uv (or a justified equivalent), records locked-install commands, adds scripts/test markers, and turns the planned README commands into verified instructions. No such files or installations are implied by the current skeleton.

## D-010 — Small original catalog and safe outline exports

**Status:** Accepted direction; catalog Open before Phases 2/5.

Use a few original hair/face/clothing options, a stable neutral pose, safe generated SVG, and bounded PNG output. Prefer drawing shared paths with existing Pillow capabilities; evaluate a dedicated SVG rasterizer only if curve fidelity demands it.

**Why:** bounded styles are easier to personalize, test, and print. Self-contained outlines avoid client-supplied executable markup.

**Alternatives:** arbitrary outfit reconstruction, SVG upload/editing, and exact pose copying expand security and geometry risks without proving the core workflow.

**Consequences:** unsupported details are explicit suggestions/defaults or manual choices. Official logos/prints and proprietary assets are excluded.

## Open questions and resolution gates

| Question | Resolve by | Required evidence |
| --- | --- | --- |
| Exact runtimes/packages and Python lock workflow? | Phase 1 | Repeatable Windows/Linux install and tool checks; record versions and commands. |
| Model assets and licensing/redistribution terms? | Before Phase 3 inference | Official origins, approved licenses, checksums, download/setup documentation, offline test. |
| Self-intersection method and final-scale thickness coverage? | Phase 2, before printable claims | Robust known-valid/invalid cases, supported template bounds, failure/unknown behavior, numeric tolerances. |
| Smallest useful original face/hair/outfit catalog? | Phase 2 baseline, refined Phase 5 | Actual image mapping feasibility, distinct results, and geometry at every supported size. |
| Can local heuristics suggest hair/clothing reliably enough? | Phase 3 | Representative fixture measurements and honest review/default behavior; no invented accuracy target. |
| Initial dimensions, feature limits, triangle budget, and primary printer setup? | Phase 2/7 | Slicer checks and physical trial; explicitly narrow supported profiles if needed. |
| Which reference hardware and latency targets are supportable? | Phase 6 | Recorded CPU/RAM/GPU, accepted images, stage timings, viewer performance. |
| Does PNG fidelity require another dependency? | Phase 4 | Shared-path raster output compared with SVG and a concrete dependency benefit. |
| How would a public deployment protect anonymous jobs? | Separate post-MVP hosting decision | Deployment threat/resource/retention requirements; no implied account or commerce implementation. |

The proposed defaults let implementation begin without pretending these validation questions are answered. An unresolved export-safety question blocks validated export support, not unrelated foundation work.
