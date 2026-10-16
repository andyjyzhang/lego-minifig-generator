# Geometry and printing contract

Status: proposed manufacturing requirements. No meshes have been generated, validated, sliced, or printed in this repository. Numerical targets are conservative starting points to test, not demonstrated capabilities or printer guarantees.

## STL limitations

STL describes a triangulated surface without standardized units, colors, materials, or printer settings. Binary STL is the planned output; color extensions are not part of this contract. Preview colors cannot promise a multicolor print. See the [Library of Congress STL format description](https://www.loc.gov/preservation/digital/formats/fdd/fdd000504.shtml).

The UI and companion validation report must state that STL coordinates represent millimeters. Download a GLB for color/material appearance and SVG/PNG for outlines. A slicer must receive the STL at its documented size; do not silently scale it to a printer bed or rely on file metadata for units.

## Dimensions and coordinates

| Property | Initial planned rule |
| --- | --- |
| Internal geometry and STL | Millimeters, right-handed axes: X right, Y toward the character's back, Z up; face looks toward -Y. |
| Base/origin | Centered in X/Y, lowest printable point at Z = 0. |
| Face Only size | 24 mm total height by default; proposed supported range 20–40 mm. Includes hairstyle/relief extent. |
| Full Body size | 80 mm total height by default; proposed supported range 60–120 mm. Includes hairstyle/relief extent. |
| Scaling | Uniform target-height scaling followed by final-scale feature enforcement and validation; no unchecked arbitrary scaling. |
| GLB | Meters, right-handed, Y up. Apply `(x, y, z)_mm -> (x, z, -y)/1000`; the inverse must recover STL space. |
| Width/depth | Derived from original style ratios and reported from the final mesh bounding box; never measured from the person's real body. |

The GLB unit/axis convention follows the [glTF 2.0 specification](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units). Phase 2 must verify the exporter behavior with reload/round-trip tests to avoid double conversion.

Dimensions are original display-model proportions and do not copy official toy measurements. Exact compatibility with any commercial brick/connector system is not supported.

## Required geometry

- One closed, fused, connected solid per STL; no articulated joints or independently floating accessories.
- Finite coordinates, valid indices, nonzero faces, no zero-area or duplicate faces, and no unintended coincident surfaces.
- Every surface edge has exactly two incident faces; vertex neighborhoods must also be manifold.
- Consistent outward winding, positive enclosed volume, no open boundaries or self-intersections.
- Actual Boolean unions at head/neck/torso, arms/hands, legs/feet, hair, and raised facial details. Overlapping watertight pieces must not simply be concatenated.
- Broad enough connected regions for structural features, and a stable supported standing orientation for Full Body. A base may be added only as an explicit visible style choice if evidence requires it.
- Physical facial/clothing relief where a feature is promised in STL; material shading or texture alone is insufficient.
- Bounded triangulation: initial maximum 200,000 triangles; provisional curved-surface chord deviation target at most 0.1 mm. Measure quality/performance and record any revision.

Default models are solid. Hollowing, drain holes, trapped resin, press fits, removable parts, sockets, and articulated joints are future scope. A closed solid can still need supports and orientation work.

## Provisional minimum features

Values are defined at the final printed scale. They are design rules for this project's bounded templates, not universal material specifications. The selected profile must be recorded in the result and report.

| Feature | `fdm_standard_v1` | `resin_standard_v1` |
| --- | --- | --- |
| Intended reference setup | 0.4 mm nozzle, approximately 0.2 mm layers | Approximately 0.05 mm layers; resin/machine recorded in trial |
| General wall/structural feature thickness | At least 1.2 mm | At least 0.8 mm |
| Load-bearing small connections | At least 2.0 mm | At least 1.5 mm |
| Raised/recessed line width | At least 1.2 mm | At least 0.5 mm |
| Raised/recessed relief depth | At least 0.6 mm | At least 0.3 mm |
| Intentional open separation | At least 0.8 mm, or close/fuse it | At least 0.5 mm, or close/fuse it |

FDM is the proposed primary profile. Resin remains provisional until separately trialed. The full body is fused, so these separations are visual gaps, not mechanical clearances. Recesses must preserve underlying solid thickness; acute feature tips must be rounded or truncated rather than approaching zero thickness. Count connection cross-sections, not merely overlap depth.

Small eyes, eyebrows, hair strands, narrow neck/wrist regions, and clothing relief are likely failure points. Prefer simplified chunky shapes, shallow safe relief, and broad joins. If a requested scale/option combination violates limits, reject it or propose a clearly disclosed simplification. Do not silently remove personalized details while reporting the original parameter set as satisfied.

## Validation approach

The proposed validator is a gate, not a mesh beauty score:

1. Validate parameter ranges, supported profiles/styles, numeric finiteness, and expected dimensions.
2. Validate shared 2D profiles: valid rings/polygons, allowed winding/holes, no self-crossings, and feature dimensions before extrusion/solid operations.
3. Generate bounded known templates; check analytical feature dimensions, connector cross-sections, relief depth, and remaining thickness at the requested scale.
4. Perform checked Boolean operations with Manifold3D or another explicitly approved solid engine; require success status and document numeric tolerances.
5. Audit the final mesh for finite/nondegenerate data, edge/vertex manifoldness, winding, positive volume, connected components, and self-intersections using a verified method.
6. Apply final-mesh thickness/intersection checks appropriate to the supported templates. Use conservative local sampling only alongside analytical template constraints; explicitly report coverage and limitations. A skipped required check has status `unknown`, which blocks STL.
7. Serialize binary STL, reload the exact bytes, weld only equivalent serialized positions within a documented tolerance, and rerun required checks. Check bounds, topology, volume, features, and intersections after serialization rather than trusting the pre-export mesh.
8. Export GLB from the same final mesh; round-trip its scale/axes and compare to STL using a documented geometric tolerance. Material boundaries may duplicate vertices but must not change the surface.
9. Publish the report and STL atomically only if all required checks pass. Revalidate after repair; never substitute slicer auto-repair for application validation.

[Trimesh documents separate watertightness, winding, and volume checks](https://trimesh.org/trimesh.html#trimesh.Trimesh.is_volume); these do not by themselves establish minimum thickness or absence of every self-intersection. The exact robust intersection/thickness method is an open Phase 2 decision in [DECISIONS.md](DECISIONS.md). Until that decision is implemented and verified, no model can be described as printable.

For initial round-trip tests, propose a 0.01 mm maximum surface/bounds deviation and 0.1% relative volume deviation between equivalent serialized outputs; record the comparison method and tighten/adjust with evidence. These comparison tolerances do not replace profile feature limits or topology checks. Repair/welding tolerances must remain far below supported feature/gap sizes.

## Report and download contract

A JSON report must include:

- Parameter schema version, immutable revision ID, generator version, print-profile ID/version, and validation implementation version.
- Final STL SHA-256 checksum, triangle/component counts, XYZ bounds in mm, volume in mm³, and timestamps.
- Each required check with `passed`, `failed`, or `unknown`, its measured/allowed values, and relevant limitations.
- Overall `passed` only when every required check passed; otherwise `failed` or `unknown`, with no STL artifact available.
- Safe repair history, advisories about supports/orientation, and geometric comparison results for preview/export consistency.

Geometry validation is server-side. The frontend also gates controls but cannot bypass it through a direct download URL. A failed/unknown result may expose a review-only GLB and report that prominently say invalid/unverified. All outputs expire with their anonymous job. A newer draft makes an old valid STL stale in the UI even if it remains valid for its own original revision.

## FDM versus resin

FDM needs attention to nozzle-sized details, overhangs, layer orientation, bridges, supports, and weak layer-direction connections. Hands/hair/face relief should use simplified supported features. Do not promise support-free printing or treat a single overhang angle as universal.

Resin can capture finer details but thin connections can be brittle, supports can scar facial surfaces, and exposure/post-curing affect final dimensions. Hollow resin parts require drainage/venting design and are excluded. Profile validation does not generate machine-specific support structures or curing instructions.

## Verification and known limitations

Phase 2 must inspect one head and one full-body STL in a slicer at the expected dimensions, with auto-repair disabled or its intervention reported as a validation failure. Phase 7 repeats this on final representative exports, then physically prints at least one model using the declared primary profile. Record machine/material, orientation, layer/nozzle settings, supports, measured dimensions, feature survival, photographs with no private source images, and failures.

A passed report establishes conformance to implemented geometric checks for one profile. It does not certify durability, safety, exact likeness, manufacturing tolerances, food contact, child suitability, or universal printer success. Small features, extreme likeness requests, hidden photo details, and unsupported style combinations remain constrained. Any unsupported printer/material combination must be described as unverified.
