FROM ros:humble-ros-base-jammy

ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_DISTRO=humble

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-humble-desktop \
    ros-humble-ros-gz \
    ros-humble-joy-linux \
    ros-humble-pcl-ros \
    ros-humble-depth-image-proc \
    python3-colcon-common-extensions \
    python3-vcstool \
    python3-rosdep \
    git \
    curl \
    wget \
    mesa-utils \
    x11-apps \
    xterm \
    && rm -rf /var/lib/apt/lists/*

RUN rosdep init 2>/dev/null || true

WORKDIR /workspace

CMD ["/bin/bash"]
