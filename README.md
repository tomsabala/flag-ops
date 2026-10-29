# Resilient Autonomy Lab

**An interactive robotics lab for autonomous navigation under changing conditions.**

## The idea

Configure a robot, give it a destination, let it navigate in a simulated world, and interfere while the mission is running. Add an obstacle. Degrade a sensor. Restrict mobility. Interrupt a communication link. Inspect what *actually* happened: what the robot sensed, believed, commanded, and did.

The goal is **not** to make every robot reach the destination. The goal is to study whether a robotic system can execute a navigation task, recognize when its assumptions are no longer valid, and behave appropriately—including stopping when safe navigation is no longer possible.

This project is deliberately **robotics-first**. The operator dashboard exists to configure experiments and expose robot behavior; the main engineering work is robot integration, navigation, localization, control, sensing, simulation, and fault handling.

## The user experience

An experiment follows one continuous workflow:

1. **Choose a robot.** Select an actual simulated platform type—for example, a Roomba-style differential-drive mobile robot or a car-like steering vehicle. This is not a cosmetic skin selection: the robot determines valid controls, sensors, motion constraints, and configuration options.
2. **Configure the world.** Choose a compatible environment, initial pose, relevant obstacles, and supported robot configuration. Options that a selected robot cannot physically or technically support are unavailable or clearly marked unsupported.
3. **Set the mission.** Select a reachable-looking destination in the appropriate configuration space and issue a navigation task.
4. **Observe the run.** Watch the live simulation alongside the planned route, actual motion, robot pose estimate, operational state, sensor status, commands, and telemetry.
5. **Interfere in real time.** While the mission is in progress, add or move obstacles, inject supported sensor faults, or alter supported capabilities. Multiple interventions can be applied during one run.
6. **Inspect the outcome.** Review the timeline, subsystem signals, resulting behavior, and whether the robot arrived, recovered, stopped safely, or failed.

The platform should make it easy to answer: **What changed? What did the robot perceive? Why did it respond that way? Was the response safe and physically plausible?**

## Robot selection: a core requirement

The platform must support **selecting different kinds of robots**, not just selecting a different visual model. A Roomba-like robot and a car must have appropriately different movement constraints, valid sensor sets, control capabilities, and mission feasibility.

The operator picks a **map** and a **team preset** and is matched against a randomly selected opponent preset; each robot runs its own autonomy stack, holds its own world model, and decides for itself. **The operator does not pilot any robot.** Several robots therefore run at once, in one world, in one match — and each one is still a genuinely modelled platform, not a token on a board. Nothing is coordinated through shared memory and nothing is resolved from simulator truth except by the referee.

Robot choices influence the rest of the setup: environment compatibility, configuration-space constraints, control choices, available interventions, and the dashboard signals that matter. A configuration or failure mechanism may be available for one robot and invalid for another.

We begin with ground navigation to keep the first result achievable; aerial and legged platforms can follow later **without retroactively pretending their navigation and control problems are identical**.

## MVP scope

The MVP demonstrates a complete, real robotics workflow rather than a large collection of mock features.

### Included

| Area | MVP outcome |
| --- | --- |
| **Selectable robots** | At least **two genuinely distinct simulated ground robot types**: a Roomba-style differential-drive robot and a car-like steering robot. Different motion/control constraints must affect navigation. |
| **Compatible worlds** | At least one usable simulated environment per robot, with meaningful obstacles and collision behavior. A shared environment is desirable only when appropriate for both. |
| **Mission** | Select start and destination, start navigation, monitor progress, and report a clear terminal outcome. |
| **Real navigation** | The robot's movement must arise from a working navigation/control loop and simulated motion, not a canned route animation. A newly blocked route must trigger a genuine navigation response. |
| **Localization** | At least one operational pose-estimation path whose inputs and outputs can be inspected and compared with simulator ground truth. |
| **Live interference** | Insert an obstacle during navigation **and** inject at least one supported sensor or capability fault that reaches the subsystem it is supposed to affect. Allow repeated interventions within a mission. |
| **Robot-specific configuration** | Only offer capabilities, parameters, and fault options that the selected robot and current simulation actually implement. |
| **Operator dashboard** | Robot/world/mission setup; live simulation; mission controls; real subsystem state and telemetry; interference controls; event timeline. |
| **Evidence** | A way to reproduce core scenarios and inspect their outcomes, including a baseline and a disrupted run. |
| **Match** | Two teams of robots in one world, playing Capture the Flag under a **referee** that owns flag possession, zone capture, score, match clock and the terminal outcome. The referee is the only subsystem that resolves anything from simulator truth. |
| **Per-robot world model** | Each robot holds its own beliefs — own pose estimate, enemy and teammate tracks with their age and their source. Two teammates must be able to disagree about the same enemy; a shared blackboard is a fake. |
| **Modeled radio link** | Teammate-to-teammate messages delivered over a declared link with range, path loss, wall attenuation, packet loss, latency and jamming. Message **content** is drawn only from the sender's own knowledge, never from truth. |
| **IFF perception** | A per-robot sensor that reports which other bodies are visible, within a stated field of view and range, with occlusion tested against the same geometry the robots navigate. Range and bearing in the sensor frame only; the robot converts to the map with its own estimate. |

The first implementation can complete the full workflow with one robot before integrating the second, but **the MVP is not complete until both robot types actually navigate under their own constraints**. `ral_car` is **parked** out of the CTF match path — it stays buildable, tested and evidenced (`docs/car-closed-loop.md`, `docs/repeatability-baseline_car.md`, the `narrow_doorway_car` / `narrow_doorway_roomba` pair), and no ArduRover SITL process runs inside a match.

### Explicitly outside the MVP

- Physical robot deployment and hardware-in-the-loop.
- Aerial or marine platforms.
- Human-in-the-loop piloting: the operator configures and disturbs a match, and never drives a robot.
- Networked play across hosts; a match runs in one simulator on one machine.
- Arbitrary robot uploads or universal compatibility with every manufacturer.
- General-purpose manipulation, object recognition, or autonomous exploration.
- A custom physics engine, custom replacement for established robotics stacks, or a full-featured commercial ground-control station.
- A promise that a robot always recovers or always reaches its destination.
- Every conceivable failure mode, sensor, protocol, or robot category.

## Representative experiments

**Baseline navigation** — A selected robot reaches a goal in a compatible environment. Inspect the computed route, pose estimate, commanded motion, actual motion, and outcome.

**Unexpected obstacle** — Insert a solid obstacle across the planned path during navigation. The simulated robot must not pass through it. Observe whether the navigation system avoids, replans, waits, or concludes that the goal is unreachable.

**Degraded sensing** — During a run, inject a fault into a sensor that the selected robot's localization or navigation system *actually uses*. Verify the change in sensor data, the downstream estimate or decisions, and the final motion/safety behavior. The correct outcome may be recovery, degraded operation, or a safe stop.

**Different robot, same objective** — Run a comparable start-to-goal task with a Roomba-style robot and a car-like robot. Show that different motion constraints produce meaningfully different commands, paths, or feasible maneuvers.

## What the dashboard must reveal

The dashboard is an **engineering instrument**, not just a visual control panel. Where a signal genuinely exists, expose:

- Robot identity, active capabilities, configuration, and health.
- Mission request, navigation status, current target, and outcome.
- Simulation view with robot geometry, obstacles, goal, and planned/actual path.
- **Ground-truth pose vs. estimated pose**, clearly distinguished.
- Sensor status and relevant raw/processed observations or their summarized measurements.
- Motion commands vs. measured movement, where available.
- Fault injection state and its actual effect on the subsystem.
- Timestamped system events and error/recovery transitions.
- A small set of computed experiment metrics with defined measurement sources.

Do not manufacture numbers or claim a subsystem is observable before it is instrumented. An honest “not supported/not measured” is better than a convincing fake.

## What success looks like

The MVP is complete when a reviewer can:

1. Select either of **two distinct robot types** and see the configuration/constraints change accordingly.
2. Start a real simulated navigation mission for each robot and observe robot-specific movement.
3. Introduce an obstacle mid-run and observe a physical, navigation-relevant consequence.
4. Introduce at least one genuine supported sensing or capability fault mid-run and trace its effect from injection to robot behavior (or documented lack of effect, if expected).
5. Inspect a meaningful discrepancy between estimated state and ground truth when relevant, along with real commands, telemetry, and mission events.
6. Re-run a baseline and disrupted scenario and explain the observed behavior using evidence rather than scripted UI messages.
7. Observe an explicit unsuccessful or safe-stop outcome when navigation cannot be completed safely.

**A failed mission can be a successful experiment.** Clear, defensible evidence matters more than a visually successful demo.

## Engineering priorities

Prioritize robotics substance over dashboard breadth:

1. Realistic simulated dynamics and robot-specific movement limits.
2. Working robot integration, navigation, localization, and control.
3. Faults that propagate to real robotic inputs/behavior.
4. Traceable telemetry, diagnostics, and testable outcomes.
5. A clear operator experience that makes the engineering accessible.

Use established robotics software and platform-appropriate protocols where they make sense. Interoperability with systems such as ROS 2, MAVLink, or vendor interfaces is relevant, but **no single protocol should be imposed on robots it does not naturally serve**. Exact software, component, and architecture choices remain open.

## Repository guidance

- [`ENGINEERING.md`](ENGINEERING.md) — Non-negotiable realism, integration, observability, and validation standards.
- [`ROADMAP.md`](ROADMAP.md) — the audit of the inherited repository and the phase plan P0–P9 that turns it into Capture the Flag. Every claim in it is labelled MEASURED, READ, UNVERIFIED or OPEN.
- [`DASHBOARD_UI_SPEC.md`](DASHBOARD_UI_SPEC.md) — Requirements and constraints for the operator dashboard: the data contract, provenance rules, panels, states and acceptance checks. Its single-robot framing is superseded in part; see the status banner at the top of that file.
- [`CLAUDE.md`](CLAUDE.md) — Working instructions for the coding agent; the agent must preserve this project's intent and the engineering standards.

**Status:** Two robots are integrated and measured, each under its own autonomy
stack. `ral_roomba` is a differential-drive platform driven by Nav2 and
localized by AMCL against the committed map; `ral_car` is an Ackermann vehicle
driven by ArduRover SITL over MAVLink/MAVROS and localized by the autopilot's
own EKF3, with no Nav2 and no AMCL. Which stack a scenario brings up, which gz
model it spawns, what its estimate IS and which faults are real for it all come
from one file per platform, `ros2_ws/src/ral_bringup/config/robots/<robot>.yaml`.

**`ral_car` is parked out of the CTF match path.** It stays buildable, tested
and evidenced — `scripts/checks/car_nav_check.sh` returning 13/13 is what
proves the park is intact, and every phase re-runs it — but no ArduRover SITL
process runs inside a match, and the CTF work below is Roomba-only until a
second match-capable platform exists.

**The CTF conversion is staged as `ROADMAP.md` P0–P9.** Nothing in the match
path — referee, radio, IFF perception, weapons, the decision model — exists
until its phase lands with its own measured evidence document.

Measured evidence, each file generated by the harness that produced it:

- [`docs/nav-closed-loop.md`](docs/nav-closed-loop.md) — the Roomba's map, AMCL
  and Nav2 loop.
- [`docs/car-closed-loop.md`](docs/car-closed-loop.md) — the car's SITL, MAVROS
  and EKF3 loop, with its limitations stated.
- [`docs/repeatability-baseline_roomba.md`](docs/repeatability-baseline_roomba.md)
  and [`docs/repeatability-baseline_car.md`](docs/repeatability-baseline_car.md)
  — the spread of each baseline over three runs.
- [`docs/gate.md`](docs/gate.md) — the rendering/performance gate and the
  lockstep measurement the car's clock depends on.

The two-robot acceptance scenario is the `narrow_doorway_roomba` /
`narrow_doorway_car` pair: one mission, one world, identical poses. MEASURED,
after the car's rear-wheel actuator was resized to make its configured 1.0 m/s
reachable and its aim point recentred between the two gaps: **both platforms
complete it**. The Roomba plans around the geometry and finishes 0.11 m from
the goal; the car, which has no planner and drives straight at its target,
threads wall A at y = 4.10 and wall B's 0.70 m doorway at y = 3.07 with no
contact at all, and stops 1.40 m short of the goal — the resolution of its own
arrival rule, not an obstacle.

That is a measured result that contradicts the earlier expectation, which was
that the car would stall against wall B; the earlier run did, with a 35 rad/s
actuator and an aim point 0.10 m off the gap centre. The platforms are still
genuinely different, and the difference is measured rather than asserted: the
car cannot rotate in place, carries a 0.611 m minimum turning radius
(`car_nav_check` check 9 measures what each mission actually demanded), has no
path planner and no obstacle avoidance, and stops where a quantised `wp_dist`
plus an estimate-side gate let it. The doorway was NOT narrowed to restore the
old outcome: `ral_apartment.sdf` is the world both robots' committed maps and
baselines belong to.

Of the five faults, the car supports four; `scan_degradation` is REFUSED for it
with a stated reason, because nothing in its stack consumes the lidar.

The **operator dashboard** is live. `scripts/dashboard.sh` starts a long-lived
gateway (`ros2 run ral_eval dashboard_api`) inside the container and serves the
console at <http://localhost:8771>. From the browser an operator starts and
stops the stack for either robot, watches bring-up topic by topic, sets a goal
in the gz world frame, runs a mission, injects any fault the selected robot
supports, and reads the terminal outcome — then switches to the Run Inspector
for the committed evidence under `runs/`.

Three properties are structural rather than stylistic. A browser mission is
driven through the **same** `MissionNode` and `MissionDriver` a measured run
uses, so it takes the identical path and ends with the identical vocabulary; it
deliberately writes **no run directory**, because `runs/` is the record of
`run_scenario` output and the console says so. The browser link is labelled
**"dashboard link"** everywhere and is JSON over SSE + POST, not a robotics
protocol. And `scripts/dashboard.sh` takes the same `flock` on
`/tmp/ral-run-$ROS_DOMAIN_ID.lock` that `scripts/run_scenario.sh` does, so a
scenario run and the dashboard can never drive one domain.

The browser half lives in `dashboard/` (React + Vite, fonts bundled, no CDN).
Build it with `cd dashboard && npm install && npm run build`; the gateway serves
`dist/` and says so when it is absent. `VITE_RAL_SOURCE=mock npm run dev` runs
the console against a recorded fixture with no container and no stack, badged
`MOCK`.
