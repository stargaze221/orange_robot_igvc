# Orange Robot IGVC — TNTech Simulation

This repository contains the Orange Robot ROS 2 stack together with the TNTech IGVC simulation workflow.

The current simulation baseline targets:

- Ubuntu 22.04
- ROS 2 Humble
- Gazebo Fortress
- Docker Compose
- NVIDIA GPU acceleration
- X11 GUI forwarding for Gazebo, RViz, and image tools

Simulation and real-robot runtimes are intentionally kept separate while sharing compatible ROS 2 topic and TF interfaces.

## Simulation Quick Start

From the repository root, load the helper functions:

```bash
source tools/igvc_shell.sh
```

Then open a simulation shell with temporary X11 access:

```bash
igvc-gui
```

The helper grants X11 access only to the local root user used by the current simulation container and revokes that access when the shell exits. This avoids the common Gazebo startup error:

```text
Authorization required, but no authorization protocol specified
qt.qpa.xcb: could not connect to display
```

For a non-GUI simulation shell, use:

```bash
igvc-sim
```

### First Build or After Source Changes

Inside the simulation container:

```bash
cd /workspace

colcon build \
  --symlink-install \
  --packages-up-to orange_gazebo orange_teleop

source install/setup.bash
```

### Launch the IGVC Baseline

```bash
ros2 launch orange_gazebo orange_igvc_baseline.launch.xml
```

The current sensor-contract branch includes the validated mobile-robot baseline together with:

- Hokuyo LiDAR
- front downward RGB camera
- front co-located depth camera

Current front camera topics include:

```text
/camera_front/color/image_raw
/camera_front/color/camera_info
/camera_front/depth/image_raw
/camera_front/depth/camera_info
```

## Detailed Documentation

See:

- [`docs/SIM_BASELINE.md`](docs/SIM_BASELINE.md) — reproducible Docker / ROS 2 / Gazebo baseline and validation history
- [`docs/IGVC_2026_COURSE_SPEC.md`](docs/IGVC_2026_COURSE_SPEC.md) — deterministic 2026 IGVC course specification
- [`orange_ros2/README.md`](orange_ros2/README.md) — upstream Orange ROS 2 package documentation

## Development

Current simulation sensor development is performed on:

```text
hyoon/sim-sensor-contract
```

Simulation changes should be validated in Docker before being proposed for merge into the main project branch.
