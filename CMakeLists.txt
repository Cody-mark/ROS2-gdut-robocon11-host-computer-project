cmake_minimum_required(VERSION 3.24)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(robot_vision)

# ---- 查找依赖 ----
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(OpenCV REQUIRED)
find_package(onnxruntime REQUIRED)

# ---- onnxruntime 运行时库路径 ----
get_filename_component(onnxruntime_LIB_DIR "${onnxruntime_LIBRARY}" DIRECTORY)

# ---- asio 头文件库 ----
add_library(asio INTERFACE)
target_include_directories(asio INTERFACE ${CMAKE_SOURCE_DIR}/asio/include)

# ---- 公共头文件 ----
file(GLOB INCLUDE_FILES "include/*.hpp")

# ---- weapon 检测节点 ----
add_executable(weapon_node
  ${INCLUDE_FILES}
  src/weapon_node.cpp)

ament_target_dependencies(weapon_node PUBLIC rclcpp std_msgs)

target_include_directories(weapon_node PUBLIC
  include/
  ${OpenCV_INCLUDE_DIRS}
)

target_link_libraries(weapon_node PUBLIC
  ${OpenCV_LIBS}
  onnxruntime::onnxruntime
  asio
  $<$<BOOL:${WIN32}>:ws2_32>
)

target_compile_definitions(weapon_node PUBLIC
  $<$<BOOL:${WIN32}>:_WIN32_WINNT=0x0A00>
)

set_target_properties(weapon_node PROPERTIES
  BUILD_RPATH "${onnxruntime_LIB_DIR}"
  INSTALL_RPATH "${onnxruntime_LIB_DIR}"
)

# ---- pole 检测节点 ----
add_executable(pole_node
  ${INCLUDE_FILES}
  src/pole_node.cpp)

ament_target_dependencies(pole_node PUBLIC rclcpp std_msgs)

target_include_directories(pole_node PUBLIC
  include/
  ${OpenCV_INCLUDE_DIRS}
)

target_link_libraries(pole_node PUBLIC
  ${OpenCV_LIBS}
  onnxruntime::onnxruntime
  asio
  $<$<BOOL:${WIN32}>:ws2_32>
)

target_compile_definitions(pole_node PUBLIC
  $<$<BOOL:${WIN32}>:_WIN32_WINNT=0x0A00>
)

set_target_properties(pole_node PROPERTIES
  BUILD_RPATH "${onnxruntime_LIB_DIR}"
  INSTALL_RPATH "${onnxruntime_LIB_DIR}"
)

# ---- 安装 ----
install(TARGETS weapon_node pole_node
  DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY
  include/
  DESTINATION include/${PROJECT_NAME})

install(DIRECTORY
  model/
  DESTINATION share/${PROJECT_NAME}/model)

install(DIRECTORY
  launch/
  DESTINATION share/${PROJECT_NAME}/launch)

ament_package()
