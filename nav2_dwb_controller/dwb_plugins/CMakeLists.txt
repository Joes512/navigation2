cmake_minimum_required(VERSION 3.5)
project(dwb_plugins)

# Find OpenMP
find_package(OpenMP REQUIRED)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(nav2_common REQUIRED)
find_package(angles REQUIRED)
find_package(dwb_core REQUIRED)
find_package(nav_2d_msgs REQUIRED)
find_package(nav_2d_utils REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(nav2_util REQUIRED)

nav2_package()

set(dependencies
  angles
  dwb_core
  nav_2d_msgs
  nav_2d_utils
  pluginlib
  rclcpp
  nav2_util
)

include_directories(
  include
)

add_library(standard_traj_generator SHARED
            src/standard_traj_generator.cpp
            src/limited_accel_generator.cpp
            src/kinematic_parameters.cpp
            src/xy_theta_iterator.cpp)
ament_target_dependencies(standard_traj_generator ${dependencies})

# Use ament_target_dependencies for linking
ament_target_dependencies(standard_traj_generator ${dependencies})

# Add OpenMP specific settings
target_link_libraries(standard_traj_generator
  ${OpenMP_CXX_LIBRARIES}
)
target_compile_options(standard_traj_generator PRIVATE ${OpenMP_CXX_FLAGS})

target_include_directories(standard_traj_generator PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  set(ament_cmake_copyright_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()

  find_package(ament_cmake_gtest REQUIRED)
  add_subdirectory(test)
endif()

install(TARGETS standard_traj_generator
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
)
install(DIRECTORY include/
        DESTINATION include/
)
install(FILES plugins.xml
        DESTINATION share/${PROJECT_NAME}
)

ament_export_include_directories(include)
ament_export_libraries(standard_traj_generator)
pluginlib_export_plugin_description_file(dwb_core plugins.xml)

ament_package()
