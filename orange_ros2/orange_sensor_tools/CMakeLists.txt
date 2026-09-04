cmake_minimum_required(VERSION 3.8)
project(orange_sensor_tools)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)

# 모든 의존성 패키지 로드 
set(ROS_DEPS
  rclcpp std_msgs sensor_msgs nav_msgs geometry_msgs laser_geometry
  tf2 tf2_ros tf2_geometry_msgs pcl_ros pcl_conversions serial
)
foreach(dep ${ROS_DEPS})
  find_package(${dep} REQUIRED)
endforeach()
find_package(PCL REQUIRED)

include_directories(include)

# --- 완전 무결 자율 추적 매크로 ---
macro(indestructible_link target)
  foreach(dep ${ARGN})
    # 1. 패키지가 실제로 발견된 깊숙한 원본 설치 경로를 실시간으로 역추적합니다.
    if(DEFINED ${dep}_DIR)
      get_filename_component(PKG_ROOT "${${dep}_DIR}/../../.." ABSOLUTE)
      message(STATUS "[자율 추적] ${dep} 패키지 경로 주입: ${PKG_ROOT}")
      
      # 해당 패키지 전용 물리적 헤더/라이브러리 경로 강제 연결
      if(EXISTS "${PKG_ROOT}/include")
        target_include_directories(${target} PUBLIC "${PKG_ROOT}/include")
      endif()
      if(EXISTS "${PKG_ROOT}/include/${dep}")
        target_include_directories(${target} PUBLIC "${PKG_ROOT}/include/${dep}")
      endif()
      if(EXISTS "${PKG_ROOT}/lib")
        target_link_directories(${target} PUBLIC "${PKG_ROOT}/lib")
      endif()
    endif()

    # 2. 혹시 모를 내부 변수 연결
    if(DEFINED ${dep}_INCLUDE_DIRS)
      target_include_directories(${target} PUBLIC ${${dep}_INCLUDE_DIRS})
    endif()
    if(DEFINED ${dep}_LIBRARIES)
      target_link_libraries(${target} PUBLIC ${${dep}_LIBRARIES})
    endif()
    if(DEFINED ${dep}_TARGETS)
      target_link_libraries(${target} PUBLIC ${${dep}_TARGETS})
    endif()

    # 3. 최신 타겟 연결
    if(TARGET ${dep}::${dep})
      target_link_libraries(${target} PUBLIC ${dep}::${dep})
    endif()
  endforeach()
endmacro()

# --- laserscan_multi_merger ---
add_executable(laserscan_multi_merger src/laserscan_multi_merger.cpp)
indestructible_link(laserscan_multi_merger ${ROS_DEPS})
target_include_directories(laserscan_multi_merger PUBLIC ${PCL_INCLUDE_DIRS})
target_link_libraries(laserscan_multi_merger PUBLIC ${PCL_LIBRARIES})
install(TARGETS laserscan_multi_merger DESTINATION lib/${PROJECT_NAME})

# --- led_node ---
add_executable(led_node src/led_node.cpp)
indestructible_link(led_node rclcpp std_msgs serial)
install(TARGETS led_node DESTINATION lib/${PROJECT_NAME})

# --- wheel_imu_odom ---
add_executable(wheel_imu_odom src/wheel_imu_odom.cpp)
indestructible_link(wheel_imu_odom rclcpp sensor_msgs nav_msgs geometry_msgs tf2 tf2_ros tf2_geometry_msgs)
install(TARGETS wheel_imu_odom DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY config firmware include launch src DESTINATION share/${PROJECT_NAME}/)

ament_package()
