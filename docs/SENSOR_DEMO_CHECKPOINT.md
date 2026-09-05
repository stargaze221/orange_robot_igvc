# IGVC Sensor Demo Checkpoint

**Checkpoint date:** September 5, 2026  
**Development branch:** `hyoon/sim-sensor-contract`

This document records the student-shareable ROS 2 / Gazebo sensor-demo state that was manually validated before moving on to higher-level perception and navigation work.

## Launch

From the repository root on the host:

```bash
source tools/igvc_shell.sh
igvc-gui
```

Inside the container:

```bash
source /workspace/install/setup.bash
ros2 launch orange_gazebo orange_sensor_demo.launch.xml
```

After source changes, rebuild with:

```bash
cd /workspace
colcon build \
  --symlink-install \
  --packages-up-to orange_gazebo orange_teleop
source install/setup.bash
```

## Verified Demo Experience

The single `orange_sensor_demo.launch.xml` entry point prepares:

- Gazebo Fortress with the deterministic IGVC course and Orange robot
- automatic third-person Gazebo follow / chase camera
- keyboard teleoperation in a separate xterm window
- RViz with fixed frame `odom`
- Orange robot model
- Hokuyo LaserScan visualization
- ZED forward depth point cloud
- front RealSense-like RGB image
- ZED RGB image
- rear RealSense-like RGB image available in RViz but disabled by default

The resulting layout is intended to be used directly for student demonstrations and short screen-recorded checkpoint videos.

## Competition-Aligned Camera Layout

The simulation camera roles and poses follow the current `LCAS-Lab/TnTechIGVC2026` `Competition` branch.

### Front RealSense-like RGB-D camera

Purpose: downward lane sensing and near-field depth.

```text
pose relative to base_link:
  xyz = (0.42, 0.0, 1.48) m
  rpy = (0.0, 1.2915, 0.0) rad

resolution = 424 x 240
rate       = 15 Hz
depth      = 0.10 to 2.50 m
```

ROS topics:

```text
/camera_front/color/image_raw
/camera_front/color/camera_info
/camera_front/depth/image_raw
/camera_front/depth/camera_info
```

Optical frame:

```text
camera_front_optical_frame
```

### Rear RealSense-like RGB camera

Purpose: rear/downward lane sensing. The current Competition configuration uses RGB only.

```text
pose relative to base_link:
  xyz = (-0.42, 0.0, 1.00) m
  rpy = (0.0, 1.2915, 3.14159) rad

resolution = 424 x 240
rate       = 6 Hz
```

ROS topics:

```text
/camera_rear/color/image_raw
/camera_rear/color/camera_info
```

Optical frame:

```text
camera_rear_optical_frame
```

### ZED-like forward RGB-D camera

Purpose: forward perception, obstacles, and future NVBlox integration.

```text
mount pose relative to base_link:
  xyz = (0.43, 0.0, 1.58) m
  rpy = (0.0, 0.43, 0.0) rad

resolution = 1280 x 720
rate       = 15 Hz
depth      = 0.20 to 5.00 m
```

ROS topics are aligned with the Competition ZED lane/perception code:

```text
/zed/zed_node/rgb/image_rect_color
/zed/zed_node/rgb/camera_info
/zed/zed_node/depth/depth_registered
/zed/zed_node/depth/camera_info
```

Optical frame:

```text
zed_left_camera_optical_frame
```

## ZED Point-Cloud Path

The RViz forward depth cloud is intentionally reconstructed on the ROS side:

```text
Gazebo ZED depth camera
  -> ros_gz_bridge
  -> /zed/zed_node/depth/depth_registered
  -> /zed/zed_node/depth/camera_info
  -> depth_image_proc point_cloud_xyz_node
  -> /zed/zed_node/depth/points
  -> RViz
```

This preserves the standard ROS optical-frame convention. Gazebo-native camera point clouds are not used for this demo because their XYZ convention can be ambiguous when paired with a ROS optical-frame header.

## LiDAR

Existing LiDAR simulation is intentionally retained at this checkpoint. The validated Hokuyo interface remains:

```text
/hokuyo_scan
```

LiDAR changes are outside the scope of this camera-alignment checkpoint.

## RViz Demo Defaults

The committed `sensor_demo.rviz` configuration shows by default:

```text
RobotModel                 ON
Hokuyo LaserScan           ON
ZED Forward Depth Cloud    ON
Front RealSense RGB        ON
ZED RGB                    ON
Rear RealSense RGB         OFF
TF                          OFF
```

This keeps the default demo informative without overloading the display.

## Checkpoint Boundary

This checkpoint establishes the sensor simulation and visualization contract only. The following are intentionally deferred to later work:

- lane-detection integration
- obstacle-processing nodes
- NVBlox
- SLAM
- Nav2
- autonomous course execution

Future changes should preserve these ROS topic and TF contracts whenever practical so the simulation remains compatible with the real-robot perception stack.
