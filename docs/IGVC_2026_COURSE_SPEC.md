# IGVC 2026 Course Geometry Specification

## Purpose

This document defines the geometry target for normalizing the existing `orange_igvc` Gazebo course to the 2026 IGVC AutoNav rules while preserving the useful topology already present in the repository.

The immediate goal is a deterministic, reproducible baseline course. A later phase will generate seeded random variations of lane geometry and obstacle placement without violating the competition constraints.

## Source of competition constraints

Primary reference: IGVC 2026 Competition Details, Rules and Format

- https://www.igvc.org/2026rules.pdf

Relevant AutoNav constraints are converted below to SI units for simulation.

| Quantity | 2026 rule | Simulation constraint |
| --- | ---: | ---: |
| Approximate course area | 120 ft x 100 ft | 36.576 m x 30.480 m |
| Approximate course length | 500 ft | 152.4 m |
| Track width | 10-20 ft | 3.048-6.096 m |
| Minimum turning radius | 5 ft | 1.524 m |
| Boundary-line width | about 3 in | 0.0762 m |
| Maximum ramp gradient | 15% | <= 0.15 |
| Minimum obstacle passage width | 5 ft | 1.524 m |
| Simulated pothole diameter | 2 ft | 0.6096 m |

The rules also explicitly allow randomized obstacle placement at left, right, and center positions before each run.

## Current-course observations

The current `orange_igvc` model is valuable because its topology already resembles the competition-style course: long straights, sinusoidal sections, repeated barrel obstacles, a ramp section, and a closed traversal path.

However, the present metric scale is larger than the 2026 target in several important ways:

- Representative inner/outer lane boundaries are separated by approximately **8.0 m**.
- The 2026 allowable lane-width range is **3.048-6.096 m**.
- Existing white boundary segments use approximately **0.127 m** width (5 in), whereas the 2026 rule specifies approximately **0.0762 m** (3 in).
- The current outer boundary footprint is approximately **42.4 m x 35.4 m**, around 16% larger than the nominal 2026 course area.
- The existing custom ramp uses approximately 0.085 rad pitch, corresponding to about an **8.5% grade**, so its gradient is already within the 15% rule limit.

Therefore, the course should not be replaced. The topology should be retained and the metric geometry normalized.

## Deterministic baseline target

### 1. Overall footprint

Target a nominal occupied footprint of approximately:

```text
36.0 m x 30.0 m
```

This remains close to the rule-of-thumb 120 ft x 100 ft area while leaving a small margin inside the Gazebo ground plane.

The existing course centerline topology should first be scaled globally by approximately:

```text
s_global ~= 0.86
```

This is a starting transform only. Lane widths must then be reconstructed independently rather than simply scaling both current boundaries.

### 2. Lane representation

Use a centerline-based representation conceptually:

```text
left_boundary(s)  = centerline(s) + 0.5 * width(s) * normal(s)
right_boundary(s) = centerline(s) - 0.5 * width(s) * normal(s)
```

This is preferable to directly scaling the existing inner and outer lines because it allows legal and reproducible lane-width variation later.

For the deterministic baseline:

```text
nominal lane width: 4.0 m
minimum:            3.2 m
maximum:            5.5 m
hard legal clamp:   3.048-6.096 m
```

The 4.0 m nominal width gives the Orange robot meaningful lateral-navigation difficulty while remaining comfortably inside the competition range.

Lane-width transitions must be gradual. Do not introduce step changes in width at segment boundaries.

### 3. Boundary tape

Use:

```text
line width = 0.0762 m
line height = 0.001-0.003 m visual geometry
```

Boundary geometry should remain visually detectable but should not create a meaningful physical curb. Collision geometry for tape is not required unless later perception experiments specifically need it.

### 4. Curvature

Maintain:

```text
minimum centerline turning radius >= 1.524 m
```

For the baseline, prefer a somewhat larger design minimum, approximately 2.0 m, so discretized line segments do not accidentally violate the rule after reconstruction.

The current sinusoidal character should be preserved.

### 5. Ramp

Retain the current three-part ramp concept:

```text
up-ramp -> flat section -> down-ramp
```

The existing approximately 8.5% grade is acceptable and should be retained initially.

Baseline target:

```text
ramp grade: 8-10%
hard maximum: 15%
```

Ramp location should remain fixed in the first normalized baseline so vehicle and bridge debugging are not mixed with course randomization.

### 6. Construction barrels / cones

Preserve the existing obstacle-rich layout concept, but future obstacle coordinates should be represented relative to the local course centerline rather than as unrelated global XY values.

Each obstacle placement must satisfy:

```text
at least one legal passage width >= 1.524 m
```

between the obstacle and a lane boundary.

The first normalized baseline should use fixed obstacle positions so simulation behavior is exactly reproducible.

### 7. Potholes and barricades

Do not add these to the first normalization pass.

Later variants may add:

- 0.6096 m diameter solid-white simulated potholes,
- additional competition-style obstacles where useful,
- configurable obstacle subsets.

This keeps the first milestone focused on geometry scale and vehicle operation.

## Future seeded randomization

Randomization should be a **course-generation feature**, not manual mutation of the checked-in SDF.

The generator should accept at least:

```text
course_seed
lane_variation
obstacle_variation
```

with a deterministic default seed.

Recommended design:

```text
seed = 0                 # canonical regression-test course
seed = user-selected     # alternate repeatable course
```

The same seed must always reproduce the same course for a given generator version.

### Lane randomization

Randomize slowly varying width and small centerline perturbations, not individual tape segments independently.

Suggested initial limits:

```text
nominal width         = 4.0 m
width variation       = +/- 0.6 m
smooth lateral jitter = +/- 0.15 m
hard width clamp      = [3.048, 6.096] m
```

Use low-frequency / spline-like variation so the lane remains physically plausible and does not become a jagged synthetic path.

### Obstacle randomization

The 2026 rules explicitly permit randomized left, right, and center obstacle placements. Implement this as constrained sampling in local lane coordinates.

A useful parameterization is:

```text
s_i       = longitudinal location along centerline
eta_i     = normalized lateral coordinate in lane
side_i    = left | center | right
```

Then convert `(s_i, eta_i)` into world XY coordinates from the generated centerline and normal vector.

Sampling must reject positions that violate:

- minimum legal passage width,
- collision with the ramp,
- collision with another obstacle,
- impossible placement near high-curvature boundary transitions,
- spawn / start / finish clearance zones.

### Difficulty presets

A later generator can expose reproducible presets such as:

```text
baseline  - no randomization; regression testing
easy      - wide lane, sparse obstacles
nominal   - moderate lane variation and obstacle density
hard      - narrower legal lane, denser legal obstacle placement
```

All presets must still obey the hard IGVC geometry constraints.

## Implementation sequence

1. **Preserve the existing `orange_igvc` SDF as the reference topology.**
2. Build a deterministic normalized 2026 baseline with approximately 36 m x 30 m footprint, 4.0 m nominal lane width, and 0.0762 m tape width.
3. Validate Gazebo loading, robot spawn, `/cmd_vel`, `/odom`, `/tf`, and `/joint_states` on the normalized course.
4. Visually verify robot-to-course scale and ramp traversal.
5. Only after the deterministic baseline is stable, introduce a generator with `course_seed`.
6. Add constrained barrel/cone variation.
7. Add smooth lane-width / centerline variation.
8. Add optional potholes and additional difficulty presets.

## Regression principle

The deterministic baseline remains the canonical debugging environment even after randomization is added.

Randomized courses are for robustness and perception/navigation evaluation; they should never replace the fixed baseline needed to distinguish software regressions from changed environment geometry.
