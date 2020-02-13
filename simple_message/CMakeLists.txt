cmake_minimum_required(VERSION 3.5)

project(simple_message)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(industrial_msgs REQUIRED)
find_package(rclcpp REQUIRED)

# Build static libs, to reduce dependency-chaining for industrial_robot_client
# set(ROS_BUILD_STATIC_LIBS false)
# set(ROS_BUILD_SHARED_LIBS true)

# The simple_message library is designed to cross compile on Ubuntu
# and various robot controllers.  This requires conditionally compiling
# certain functions and headers.  The definition below enables compiling
# for a ROS node.
option(ROS "Use ROS" ON)
option(LINUXSOCKETS "Use linux socket API" ON)
option(STDIOLOG "Use stdout for logging" ON)
option(FLATHEADERS "Flat headers" OFF)
option(MOTOPLUS "Build for MotoPlus" OFF)
configure_file(${PROJECT_SOURCE_DIR}/include/config.h.in ${PROJECT_BINARY_DIR}/include/config.h)

set(SRC_FILES
  ${PROJECT_BINARY_DIR}/include/config.h

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

  src/simple_comms_fault_handler.cpp)

# set(UTEST_SRC_FILES test/utest.cpp test/utest_message.cpp)

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

include_directories(
  ${PROJECT_SOURCE_DIR}/include
  ${PROJECT_BINARY_DIR}/include
  )

# add_custom_command(
#   OUTPUT  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_dummy.cpp
#   COMMAND ${CMAKE_COMMAND} -E touch ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_dummy.cpp)
# add_library(${PROJECT_NAME}_dummy ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_dummy.cpp)

# unfortunately this will have to be installed, but the linker will remove it
# from the library dependencies of dependent targets, as it contains no symbols
# that can be imported from it.

# install(TARGETS ${PROJECT_NAME}_dummy DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

# NOTE: All test files require TEST_PORT_BASE to be defined.  Defining different
# ports for each test executable allows them to run in parallel.

# DEFAULT LIBRARY (SAME ENDIAN)
add_library(simple_message SHARED ${SRC_FILES})

ament_target_dependencies(simple_message
  industrial_msgs
  rclcpp
  )

# ALTERNATIVE LIBRARY (DIFFERENT ENDIAN)
# add_library(simple_message_bswap SHARED ${SRC_FILES})
# set_target_properties(simple_message_bswap PROPERTIES COMPILE_DEFINITIONS "BYTE_SWAPPING")

# ALTERNATIVE LIBRARY (64-bit floats)
# add_library(simple_message_float64 ${SRC_FILES})
# set_target_properties(simple_message_float64 PROPERTIES COMPILE_DEFINITIONS "FLOAT64")
# target_link_libraries(simple_message_float64 rclcpp)
# add_dependencies(simple_message_float64 ${industrial_msgs_EXPORTED_TARGETS})

# install(
#     TARGETS simple_message simple_message_bswap simple_message_float64 
#     ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#     RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})

# install(
#     DIRECTORY include/${PROJECT_NAME}/
#     DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

ament_export_libraries(${PROJECT_NAME})
ament_export_include_directories(include)
ament_export_dependencies(
  industrial_msgs
  rclcpp)

install(TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION lib)
install(DIRECTORY include/simple_message
  DESTINATION include)
install(FILES
  ${PROJECT_BINARY_DIR}/include/config.h
  DESTINATION
  include
  )

ament_package()
