# Product requirements

Status: proposed MVP requirements; no functionality is implemented. Implementation progress belongs in [PLANS.md](../PLANS.md). This document defines intended product behavior, not a record of achieved capabilities.

## Problem statement

People want a recognizable, playful character based on a personal photograph and a model they can actually print. A visually convincing render can hide thin features, disconnected surfaces, incorrect dimensions, or an invalid mesh. The product must connect photo-derived personalization to constrained geometry and inspectable export validation.

A single image cannot reveal accurate hidden geometry or guarantee likeness. The MVP offers an editable stylization with honest uncertainty, a fixed original toy-like vocabulary, and documented manufacturing constraints.

## Target users

- Hobbyists with access to an FDM or resin printer who want a personalized display figure.
- Makerspaces and educators demonstrating photo analysis, procedural design, and 3D printing.
- Casual creators who want an outline or 3D character to download and customize without learning CAD.

The initial operating environment is local development or a controlled demonstration. Public multi-user hosting is not an MVP acceptance requirement.

## Product goals and success criteria

1. Complete both Face Only and Full Body journeys using actual local image analysis.
2. Preserve visible personal characteristics through bounded parameters, understandable suggestions, and simple correction controls.
3. Deliver matching outlines, preview, and downloadable geometry with useful validation evidence.
4. Keep personal photos temporary and inference free of paid AI API requirements.
5. Keep the system small enough to understand and run locally while retaining clear module boundaries.

Success is demonstrated by the [MVP definition of done](../PLANS.md#mvp-definition-of-done). Downloads alone are insufficient evidence: representative images must produce personalized results, invalid meshes must be blocked, and a representative print trial must substantiate the primary printer profile.

## Core user journeys

### Face Only

1. Choose Face Only and read guidance for one well-lit, mostly frontal, unobscured face.
2. Upload a JPEG/PNG and see its local thumbnail and privacy notice.
3. Wait through actual upload/analysis stages; recover with better input if the face cannot be used.
4. Review image-derived facial ratios/colors and hairstyle suggestions; confirm or correct uncertain features.
5. Generate a head outline and a procedural 3D head with simplified hair and physical facial relief.
6. Orbit/zoom, inspect dimensions and print geometry, and adjust supported controls.
7. Download current SVG/PNG/GLB and, when all required checks pass, STL plus its validation report.
8. Delete the session or let it expire; resubmit the photo if analysis is needed again.

### Full Body

1. Choose Full Body and read guidance for one person with a visible face, torso, arms, and legs.
2. Upload and analyze face and body evidence locally.
3. Review facial, hair, clothing-color/style, and visible body-proportion suggestions. Unsupported or hidden details require user selection or labeled defaults.
4. Generate a matching full-body outline and a neutral, fused display character. The photograph's pose is simplified into a stable supported stance.
5. Customize, inspect, validate, and download through the same revision-aware workflow.

### Failure and recovery

Bad files, ambiguous people, missing landmarks, unavailable local models, busy queues, timeouts, invalid geometry, and expired artifacts each have a clear next step. Analysis failure does not silently load a sample model. Geometry failure can retain an inspectable preview labeled invalid, but it cannot enable validated STL download. A new edit invalidates previous export eligibility until the matching revision is validated.

## Functional requirements

| ID | Requirement | Acceptance evidence |
| --- | --- | --- |
| FR-01 | Offer Face Only and Full Body with mode-specific photo guidance. | Both journeys are visible and retain their selected mode. |
| FR-02 | Accept one JPEG/PNG photo with enforced server limits. | Invalid signatures, corruption, animation, excessive bytes/pixels, and unsupported formats are rejected safely. |
| FR-03 | Analyze one usable face locally; Full Body also requires usable body evidence. | Real inference tests pass; no/multiple subjects and insufficient evidence produce actionable errors. |
| FR-04 | Map face characteristics, simplified hair/outfit suggestions, colors, and visible proportions to versioned structured parameters. | Relevant image changes alter relevant suggestions; provenance distinguishes observed, heuristic, user, and default values. |
| FR-05 | Allow correction of uncertain or unsupported appearance details. | Suggestions are editable without another upload; defaults are labeled before confirmation. |
| FR-06 | Generate face/head and full-body 2D outlines from shared parameter/profile definitions. | Current SVG/PNG matches the corresponding revision and documented front-view geometry. |
| FR-07 | Build original constrained procedural head/body solids. | Supported geometry options and size bounds pass required solid/profile checks. |
| FR-08 | Provide an interactive 3D viewer. | Orbit, zoom, dimensions, validation state, and a material-free print view work; textual status survives WebGL failure. |
| FR-09 | Support basic facial, hairstyle, outfit, color, target-size, and print-profile editing. | Edits update the current outline/GLB and revalidate affected geometry; color-only effects are clearly identified. |
| FR-10 | Download GLB, SVG, PNG, and conditionally STL with a JSON validation report. | Downloads belong to the latest confirmed revision; STL is unavailable if any required check fails or is unknown. |
| FR-11 | Show honest job states and allow restart/deletion. | Queue, failure, timeout, retry, expiry, and deletion work without simulated progress. |
| FR-12 | Remove photos after analysis and expire derived results. | Cleanup is tested on success, failure, timeout, deletion, and restart; there is no public gallery or file listing. |

## Nonfunctional requirements

| Area | Proposed requirement |
| --- | --- |
| Correctness | Same canonical parameters, generator version, and print profile produce reproducible geometry within documented numeric tolerances. |
| Geometry | Final-artifact validation, explicit units, feature constraints, true solid union, and preview agreement follow [PRINTING.md](PRINTING.md). |
| Privacy | No external photo transmission for inference, no persistent photo retention, no personal payload logging, no third-party analytics for inputs/results. |
| Security | Decode/request limits, strict schemas, non-executable SVG, protected temporary paths, high-entropy job capabilities, no-store responses, explicit CORS. |
| Resource bounds | Initial proposals: 10 MiB uploads, 20 MP decode cap, analysis dimension 2048 px, one active/four queued jobs, 120-second queue-plus-run deadline, 512 MiB temporary-disk budget. |
| Lifecycle | Delete raw/normalized images when analysis ends; expire job results and derived artifacts 3600 seconds after completion, without sliding extension. Purge prior-run data on startup. |
| Performance | Provisional target: analysis and generation each finish within 30 seconds for accepted reference images on a documented modern four-core CPU with 16 GiB RAM; hard timeout is 120 seconds. Measure before claiming support. |
| Viewer | Provisional goal: interactive orbit/zoom at 30 fps on a documented integrated-GPU desktop with bounded mesh size. Provide text/status fallback when WebGL is unavailable. |
| Accessibility | Target WCAG 2.2 AA for application controls and status, with keyboard access, visible focus, labels, contrast, reduced motion, and text equivalents for essential canvas information. |
| Compatibility | Verify Windows/Linux backend setup and current stable Chromium/Firefox desktop flows; check mobile layouts/touch interaction. Record exact tested versions at verification. |
| Maintainability | Typed API contracts, pure service boundaries, reproducible dependencies, meaningful tests, and synchronized documentation. |
| Cost | After downloading approved local models, inference and geometry generation work without paid AI APIs or outbound network access. |

Resource values are initial planning targets mirrored in [the environment example](../.env.example), not implemented controls or universal operating guarantees. Phase 1/3 must enforce them; Phase 6/7 must measure performance on named hardware and revise unsupported claims explicitly.

## MVP boundaries

Included: both modes; one photograph/person; approximate local appearance analysis; a small original style catalog; stable neutral full-body stance; solid fused models; default 24 mm head or 80 mm full-body height including hair; supported constrained scaling; outlines, preview, and validated exports; temporary anonymous sessions.

Excluded: exact reconstruction/photogrammetry, inferred real body measurements, sensitive attribute/identity inference, arbitrary photo-pose replication, arbitrary outfits/accessories, editable topology, articulation, hollowing/drainage, mating toy connectors, licensed branding, batch processing, permanent galleries, accounts, authentication, payments, shipping, and orders.

The API is designed for a controlled MVP. A public deployment requires a separate scope decision addressing abuse controls, transport, operational isolation, and deployment privacy; it must not silently grow an account/ecommerce platform.

## Release acceptance

Use representative approved photographs covering varied lighting, skin tones, hairstyles, and clothing, alongside explicit unsupported/corrupt cases. Do not retain real customer photos as fixtures.

A release requires both real end-to-end journeys, user-edit synchronization, final-STL validation failure cases, GLB/STL equivalence checks, privacy lifecycle verification, responsive/accessibility review, and documented slicer/primary-profile print evidence. If an unsupported attribute uses a default, the UI must say so. If printer access is unavailable, keep physical verification open and describe manufacturing readiness as provisional.

## Future features

Potential later work: improved local hair/clothing segmentation, more original styles, optional multi-view input, assembly-ready parts, articulation, 3MF/material-aware export, and more calibrated printer profiles. Hosted sharing or saved projects would require explicit privacy/retention requirements. Commerce and account features remain outside this project's current scope.
