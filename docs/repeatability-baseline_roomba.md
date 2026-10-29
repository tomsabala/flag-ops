# Repeatability — measured spread, not a determinism claim

Measured 2026-10-29 16:30 UTC by
`scripts/checks/repeatability_check.sh`, which runs
`scripts/run_scenario.sh baseline_roomba` 3 times in sequence
and reads the resulting `metrics.json` files. Every duration is sim time off
`/clock`.

## Per-run results

| Metric | run 1 | run 2 | run 3 |
| --- | --- | --- | --- |
| `mission_duration_s` | 37.950 | 37.384 | 37.644 |
| `path_length_m` | 13.496 | 13.518 | 13.500 |
| `goal_pose_error_m` | 0.033 | 0.012 | 0.034 |
| `est_error_mean_m` | 0.067 | 0.067 | 0.068 |
| `est_error_at_start_pose_m` | 0.081 | 0.079 | 0.074 |
| `plan_publications` | 38 | 37 | 37 |
| `recoveries` | 0 | 0 | 0 |
| `collision_count` | 0 | 0 | 0 |
| `mission_result` | `nav2:STATUS_SUCCEEDED` | `nav2:STATUS_SUCCEEDED` | `nav2:STATUS_SUCCEEDED` |
| `fault_to_response_s` | `null` (no FaultEvent reached state=activated; states seen: none) | `null` (no FaultEvent reached state=activated; states seen: none) | `null` (no FaultEvent reached state=activated; states seen: none) |

Why a value above is `null`:

- no FaultEvent reached state=activated; states seen: none

## Spread

| Metric | min | max | max − min | max − min as % of mean |
| --- | --- | --- | --- | --- |
| `mission_duration_s` | 37.384 | 37.950 | 0.566 | 1.5% |
| `path_length_m` | 13.496 | 13.518 | 0.022 | 0.2% |

## Run provenance

| Run | Image | `git_sha` | dirty | `map_sha256` | `world_to_map` | RTF |
| --- | --- | --- | --- | --- | --- | --- |
| `baseline_roomba_20261029T162416Z` | `sha256:4eb135c402d493098` | `6da2b6943eec` | True | `47d1e2ad01a41a52` | x = 0.5, y = 0.5, yaw = 0.0 | 1.0057908407657086, 1.000324605334431, 1.0028792663737591 |
| `baseline_roomba_20261029T162620Z` | `sha256:4eb135c402d493098` | `6da2b6943eec` | True | `47d1e2ad01a41a52` | x = 0.5, y = 0.5, yaw = 0.0 | 0.99862938117433819, 0.99918017266832559, 1.0019899520447608 |
| `baseline_roomba_20261029T162817Z` | `sha256:4eb135c402d493098` | `6da2b6943eec` | True | `47d1e2ad01a41a52` | x = 0.5, y = 0.5, yaw = 0.0 | 0.99883286379865133, 0.999840525436193, 0.9978033359558931 |

## What this does and does not show

- **It is a measured spread over 3 runs, not a reproducibility guarantee.** The 3 runs use the
  same scenario file, the same image and the same map; what differs is
  everything the project does not control.
- **Provenance.** Every run above records `git_dirty` in its own `env.json`; a
  `True` there means the recorded `git_sha` is the last commit, not the code that
  ran, and `git_diff_sha256` identifies the uncommitted delta. The report refuses
  to mix runs whose `image_ref`, `git_sha` or `scenario_sha256` differ.
- **`seed:` seeds only `ral_faults`' RNG** — Phase 4.2's scan noise and
  dropout. It does not seed the simulator, the planner or the localizer, so a
  baseline run with no fault is not affected by it at all.
- **`nav2_amcl` exposes no seed.** Its particle filter draws from `drand48`,
  seeded outside our control, so the estimate differs run to run even with
  identical sensor input. Every `est_error_*` number inherits that.
- **ROS callback scheduling and the 20 Hz DWB loop are not deterministic.**
  Control decisions land on slightly different sim instants each run, which
  shows up directly in `mission_duration_s` and `path_length_m`.
- **The start HEADING is aligned before the goal is sent, and it dominates**
  **everything else.** The localization warm-up is an in-place rotation, so it
  ends wherever convergence happened to land. With that arbitrary heading the
  measured spread of `mission_duration_s` over three runs was **12.5%** (71.0 to
  80.5 s): facing away from the goal, DWB reversed at 0.1 m/s for 40 s instead
  of turning. The runner now turns back to the scenario's declared initial yaw
  first, using `/amcl_pose` — the robot's own estimate, never ground truth —
  and `run.json` records `initial_yaw_aligned` and the residual AMCL believed
  it had. The reversing run was real Nav2 behaviour, but it answered a heading
  no scenario asked for.
- **The real-time factor varies with recorder I/O and host load.** It changes
  how long a run takes in wall time, not what it measures: every duration
  above is sim time.
- **A wider spread is a result, not a failure.** If a later change widens it,
  that is evidence about the change; it is not a reason to re-run until the
  numbers agree.
