# Orange Robot IGVC — Simulation Integration Fork

This repository is a fork of [`Hko44182/orange_robot_igvc`](https://github.com/Hko44182/orange_robot_igvc) used to validate, clean up, and integrate the Orange robot simulation stack before proposing changes back upstream.

## Repository roles

- **Upstream:** `Hko44182/orange_robot_igvc`
- **Integration fork:** `stargaze221/orange_robot_igvc`
- **Current working branch:** `sim-baseline-cleanup`

The goal of this branch is to establish a reproducible ROS 2 / Gazebo simulation baseline before changing research algorithms or navigation behavior.

## Main simulation stack

The current source tree is under `orange_ros2/` and includes:

- `orange_description` — robot model and sensor xacro
- `orange_gazebo` — Gazebo worlds and launch files
- `orange_sensor_tools` — sensor preprocessing and scan/cloud utilities
- `orange_slam` — SLAM launch/configuration
- `orange_navigation` — Navigation2 configuration and waypoint navigation
- `orange_teleop` — keyboard/gamepad teleoperation
- `orange_bringup` — visualization and system bringup

The IGVC simulation entry point is currently:

```bash
ros2 launch orange_gazebo orange_igvc.launch.xml
```

The launch file uses `ros_gz_sim` / `ros_gz_bridge`, so the baseline validation should be performed with the modern Gazebo integration used by the current source tree.

## Local workstation clone

Clone this fork and check out the cleanup branch:

```bash
git clone https://github.com/stargaze221/orange_robot_igvc.git
cd orange_robot_igvc
git checkout sim-baseline-cleanup
```

Add the student repository as `upstream`:

```bash
git remote add upstream https://github.com/Hko44182/orange_robot_igvc.git
git remote -v
```

Recommended remote convention:

```text
origin    -> stargaze221/orange_robot_igvc
upstream  -> Hko44182/orange_robot_igvc
```

To fetch future student changes without modifying the working branch:

```bash
git fetch upstream
```

Before proposing a pull request back upstream, update the local integration branch against the latest upstream `main` and resolve any conflicts locally.

## Baseline validation

See [`docs/SIM_BASELINE.md`](docs/SIM_BASELINE.md) for the reproducibility checklist and local workstation validation procedure.

## Development policy for this branch

1. Establish a reproducible simulation baseline first.
2. Keep functional changes separate from cleanup/documentation changes whenever practical.
3. Validate Gazebo, TF, odometry, sensors, teleoperation, and Navigation2 independently.
4. Do not treat a launch-file change as validated until it has been tested on a clean or documented workstation environment.
5. Once a change is stable, propose it back to the upstream student repository through a pull request.

## Baseline status

The current branch is **not yet declared a known-good simulation baseline**. The next step is to run the stack on a workstation and record the exact dependency, build, launch, and topic-validation sequence.
