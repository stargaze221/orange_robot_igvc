# Simulation Baseline Validation

This document defines the initial validation target for the Orange robot IGVC simulation stack. The purpose is to separate environment/integration problems from later navigation or research-algorithm changes.

## Target environment

Initial validation target:

- Ubuntu 22.04
- ROS 2 Humble
- Modern Gazebo integration through `ros_gz_sim` / `ros_gz_bridge`
- A clean ROS 2 workspace whose source is traceable to this repository and its declared dependencies

Record any deviations from this environment when testing.

## 1. Clone and configure remotes

```bash
git clone https://github.com/stargaze221/orange_robot_igvc.git
cd orange_robot_igvc
git checkout sim-baseline-cleanup

git remote add upstream https://github.com/Hko44182/orange_robot_igvc.git
git remote -v
```

Expected convention:

```text
origin    stargaze221/orange_robot_igvc
upstream  Hko44182/orange_robot_igvc
```

## 2. Record system state

Before installation/build, record:

```bash
lsb_release -a
ros2 --version 2>/dev/null || true
printenv ROS_DISTRO

git status
git log -1 --oneline
```

For Gazebo / ROS-Gazebo packages, also record the installed package versions once the environment is configured.

## 3. Dependency setup

The existing `orange_ros2/README.md` and `orange_ros2/orange_ros2.rosinstall` come from the upstream Orange ROS 2 stack and should be treated as the starting point, not yet as a fully validated recipe for this fork.

The current dependency manifest references external packages including Velodyne, Orange navigation support packages, maps, FAST_LIO, Livox simulation support, and serial support.

During the first workstation test, record the exact commands needed to:

- install ROS package dependencies,
- obtain external source dependencies,
- resolve any duplicate package names between this repository and externally fetched packages,
- build the workspace successfully.

Do not hide manual fixes. If a dependency requires a manual patch or specific commit, record it here before declaring the baseline reproducible.

## 4. Build validation

Run a clean build and record the command used. Typical ROS 2 validation should include:

```bash
colcon build --symlink-install
```

After a successful build:

```bash
source install/setup.bash
```

Baseline requirement: the build procedure must be repeatable from a clean shell using documented commands.

## 5. Launch Gazebo IGVC world

Primary simulation target:

```bash
ros2 launch orange_gazebo orange_igvc.launch.xml
```

Verify:

- Gazebo starts without fatal plugin errors.
- The Orange robot is spawned.
- Robot geometry is visually plausible.
- No critical TF or robot-state-publisher errors persist.
- Sensor plugins initialize.

## 6. Teleoperation validation

In a new sourced terminal:

```bash
ros2 launch orange_teleop teleop_keyboard.launch.xml
```

Verify:

- `/cmd_vel` is published.
- The simulated robot moves forward/backward and turns.
- Left/right wheel behavior agrees with commanded differential drive motion.
- Odometry changes consistently with motion.

## 7. Topic and TF validation

Check that the expected interface is available. At minimum inspect:

```bash
ros2 topic list
ros2 topic hz /odom
ros2 topic hz /imu
ros2 topic hz /hokuyo_scan
ros2 topic hz /mid360_PointCloud2
ros2 topic echo /joint_states --once
```

Also inspect the TF tree and verify the expected chain among frames such as:

```text
map / odom / base_footprint / base_link / sensor frames
```

Expected simulation interfaces currently include:

- `/cmd_vel`
- `/odom`
- `/fusion/odom`
- `/tf`
- `/joint_states`
- `/imu`
- `/hokuyo_scan`
- `/mid360_PointCloud2`
- `/livox_scan` after sensor processing

Not every downstream topic is required to exist immediately when only the Gazebo launch is running; record which launch step produces each one.

## 8. RViz validation

Launch the appropriate RViz configuration / bringup and verify:

- robot model,
- TF,
- odometry,
- laser scan,
- point cloud,
- fixed frame selection.

Capture at least one screenshot for the known-good baseline record.

## 9. Navigation baseline

Only after simulation, sensing, and TF are individually validated, test Navigation2.

Current Navigation2 configuration uses a conventional baseline including AMCL, NavFn global planning, and DWB local control. Before using it as a research comparator, verify that the map, scan topic, odometry topic, and initial pose are consistent with the IGVC simulation world.

Record:

- map used,
- initial pose,
- goal command / waypoint procedure,
- whether autonomous navigation reaches a simple goal,
- any parameter changes required specifically for simulation.

## 10. Known cleanup items to inspect

During validation, specifically inspect the following integration risks already visible in source:

1. Xacro link names versus references in `orange_robot_simulation.gazebo`.
2. Duplicate visual/material blocks in the Gazebo extension file.
3. Consistency of wheel radius and wheel separation between robot geometry and the differential-drive plugin.
4. Consistency of ROS-Gazebo bridge topic names and sensor plugin topic names.
5. Duplicate package/dependency risk between source already included in this repository and packages fetched by `orange_ros2.rosinstall`.
6. Navigation map defaults that still point to Hosei map assets rather than an IGVC-specific reproducible setup.

Do not change all of these at once. Reproduce each issue first, then isolate fixes into small commits.

## Definition of a known-good baseline

The branch may be tagged as a baseline only after all of the following are documented and repeatable:

- clean dependency setup,
- successful build,
- Gazebo IGVC launch,
- teleoperation,
- correct basic robot motion,
- odometry and TF,
- IMU and LiDAR data,
- RViz visualization,
- at least one documented SLAM or Navigation2 test,
- exact commit SHA used for the test.

Suggested future tag after validation:

```text
sim-baseline-v0.1
```
