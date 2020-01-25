cmake_minimum_required(VERSION 3.5)

project(simple_message)

set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
endif()

set(CMAKE_CXX_FLAGS "-Wall -Wextra -Wpedantic")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

set(SRC_FILES
  src/byte_array.cpp
  src/simple_message.cpp
  src/smpl_msg_connection.cpp

  src/socket/simple_socket.cpp
  src/socket/udp_socket.cpp
  src/socket/udp_client.cpp
  src/socket/udp_server.cpp
  src/socket/tcp_socket.cpp
  src/socket/tcp_client.cpp
  src/socket/tcp_server.cpp

  src/message_handler.cpp
  src/message_manager.cpp
  src/ping_handler.cpp
  src/ping_message.cpp
  src/joint_data.cpp
  src/joint_feedback.cpp
  src/joint_traj_pt.cpp
  src/joint_traj_pt_full.cpp
  src/joint_traj.cpp
  src/robot_status.cpp

  src/messages/joint_message.cpp
  src/messages/joint_feedback_message.cpp
  src/messages/joint_traj_pt_message.cpp
  src/messages/joint_traj_pt_full_message.cpp
  src/messages/robot_status_message.cpp

  src/simple_comms_fault_handler.cpp
)

# The simple message make file builds two libraries: simple_message and
# simple_message_byte_swapping.
#
# simple_message - is the default library.  This library should be used
# when the target for the simple message is the same endian (i.e. both
# big-endian or little-endian).  Intel based machines are little endian
#
# simple_message_byte_swapping - is an alternative library that can be used
# when the target for simple message is a DIFFERENT endian AND when the target
# target cannot perform byte swapping (as is the case for some industrial
# controllers).  This library performs byte swapping at the lowest load/unload
# levels.

# NOTE: The libraries generated this package are not included in the catkin_package
# macro because libraries must be explicitly linked in projects that depend on this
# package.  If this is not done (and these libraries were exported), then multiple
# library definitions (normal - simple_message and byteswapped - simple_message_bswap)
# are both included (this is bad).

# DEFAULT LIBRARY (SAME ENDIAN)
add_library(simple_message SHARED ${SRC_FILES})

target_include_directories(simple_message
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

# The simple_message library is designed to cross compile on Ubuntu
# and various robot controllers.  This requires conditionally compiling
# certain functions and headers.  The definition below enables compiling
# for a ROS node.

# using std io log
target_compile_definitions(simple_message PUBLIC -DSTDIOLOG)
#build using ROS libraries
target_compile_definitions(simple_message PUBLIC -DROS)
#use linux sockets for communication
target_compile_definitions(simple_message PUBLIC -DLINUXSOCKETS)

# find dependencies
find_package(industrial_msgs REQUIRED)
find_package(rclcpp REQUIRED)

ament_target_dependencies(simple_message
  industrial_msgs
  rclcpp
)

ament_export_libraries(${PROJECT_NAME})
ament_export_include_directories(include)
ament_export_dependencies(
  industrial_msgs
  rclcpp
)

install(TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION lib)
install(DIRECTORY include/
  DESTINATION include)

ament_package()
