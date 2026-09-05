# Orange Robot IGVC Simulation Baseline

This document defines the reproducible simulation environment for the Orange Robot IGVC project.

The simulation environment is intentionally separated from the real-robot runtime. Simulation and hardware should expose compatible ROS 2 topics and TF interfaces so that higher-level autonomy software can be shared.

## Architecture Decision

### Simulation

- Host OS: Ubuntu 22.04 recommended
- Container runtime: Docker Engine + Docker Compose
- ROS distribution: ROS 2 Humble
- Simulator: Gazebo Fortress
- ROS/Gazebo integration: `ros_gz`
- GPU acceleration: NVIDIA GPU through NVIDIA Container Toolkit

### Real Robot

GPU-accelerated perception, including future Isaac ROS components, is treated as a separate real-robot runtime layer.

The common interface between simulation and the real robot should be standard ROS 2 topics and TF rather than direct GPU-memory coupling.

## Initial Simulation Milestone

The first baseline is intentionally minimal:

1. Build a reproducible ROS 2 Humble + Gazebo Fortress container.
2. Confirm NVIDIA GPU access from the container.
3. Launch the Orange IGVC simulation world.
4. Spawn the robot.
5. Send `/cmd_vel`.
6. Verify `/odom`, `/tf`, and `/joint_states`.

LiDAR processing, SLAM, Nav2, and higher-level autonomy will be added only after this baseline is stable.

## Tested Host Configuration

Initial development environment:

- Ubuntu 22.04.5 LTS (Jammy)
- Docker 29.7.2
- Docker Compose v5.3.1
- NVIDIA GeForce RTX 4060
- NVIDIA driver 610.43.02

These exact versions are not strict requirements. They record the first verified development environment.

## Verified Container Baseline

The first Docker image was successfully built on September 5, 2026 using:

```bash
docker build \
  -t orange-igvc-sim:humble \
  -f docker/sim/Dockerfile .
```

The container was launched with GPU access using:

```bash
docker run --rm -it \
  --gpus all \
  orange-igvc-sim:humble
```

The following runtime checks were verified inside the container:

```bash
echo $ROS_DISTRO
ros2 pkg list | grep ros_gz
ign gazebo --versions
nvidia-smi
```

Verified results:

- `ROS_DISTRO=humble`
- `ros_gz`
- `ros_gz_bridge`
- `ros_gz_image`
- `ros_gz_interfaces`
- `ros_gz_sim`
- `ros_gz_sim_demos`
- Ignition Gazebo `6.18.0`
- NVIDIA GeForce RTX 4060 visible from inside the container

The command `gz sim` is not available in this environment. Fortress uses the `ign gazebo` CLI. This is important because the current repository contains some newer `gz`-namespace simulator conventions that may need to be normalized for the Humble/Fortress baseline.

## Host Requirements

Recommended:

- Ubuntu 22.04
- Docker Engine
- Docker Compose
- NVIDIA driver
- NVIDIA Container Toolkit

Check the host environment:

```bash
docker --version
docker compose version
nvidia-smi
```

Verify GPU access from Docker:

```bash
docker run --rm --gpus all ubuntu nvidia-smi
```

A successful result should show the host NVIDIA GPU from inside the container.

## Planned Repository Layout

```text
orange_robot_igvc/
├── docker/
│   └── sim/
│       └── Dockerfile
├── docs/
│   └── SIM_BASELINE.md
├── orange_ros2/
└── serial/
```

Docker/runtime configuration is kept outside the ROS package tree.

## Known Gazebo Migration Issue

The current repository appears to contain an incomplete Gazebo migration.

Some simulation files use modern Gazebo / `ros_gz` conventions, while some package dependencies still reference the Gazebo Classic stack through `gazebo_ros_pkgs`.

The verified Humble container installs Gazebo Fortress / Ignition Gazebo 6.18.0. Therefore, newer simulator symbols such as `gz::sim` and plugin filenames using `gz-sim-*` should be treated as potential compatibility issues until validated against Fortress.

The `hyoon/sim-baseline-cleanup` branch will normalize this simulation stack before SLAM, Nav2, and other higher-level functionality are enabled.

The intended baseline is:

```text
Ubuntu 22.04
    └── Docker
         └── ROS 2 Humble
              └── Gazebo Fortress / Ignition Gazebo 6
                   └── ros_gz
```

## Development Branch

Simulation baseline development is performed on:

```text
hyoon/sim-baseline-cleanup
```

Changes should be validated locally in Docker before being proposed for merge into `main`.
