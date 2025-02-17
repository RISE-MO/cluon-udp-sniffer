
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

# Project
project(cluon-udpsniffer LANGUAGES CXX)

# Fetch CPM
set(CPM_DOWNLOAD_VERSION 0.32.0) 
set(CPM_DOWNLOAD_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
if(NOT (EXISTS ${CPM_DOWNLOAD_LOCATION}))
  message(STATUS "Downloading CPM.cmake v${CPM_DOWNLOAD_VERSION}")
  file(DOWNLOAD https://github.com/cpm-cmake/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake ${CPM_DOWNLOAD_LOCATION})
endif()
include(${CPM_DOWNLOAD_LOCATION})

# Add dependencies
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

CPMAddPackage("gh:CLIUtils/CLI11@1.9.1")

CPMAddPackage(
  NAME cluon
  GITHUB_REPOSITORY chrberger/libcluon
  VERSION 0.0.140
  SOURCE_SUBDIR libcluon
  EXCLUDE_FROM_ALL YES
  GIT_SHALLOW YES
  PATCH_COMMAND git apply ${CMAKE_SOURCE_DIR}/external/libcluon_CMakeLists_disable_tests.patch ${CMAKE_SOURCE_DIR}/external/libcluon_UDPReceiver_interfaceAddress.patch
)


# Create executable
add_executable(${PROJECT_NAME} src/main.cpp)
target_include_directories(${PROJECT_NAME} 
  PRIVATE 
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  # cluon specific inludes
  ${cluon_SOURCE_DIR}/libcluon/include
  ${cluon_SOURCE_DIR}/libcluon/thirdparty
  ${CMAKE_CURRENT_BINARY_DIR}/include # libcluon "built-in" message specifications
)
target_link_libraries(${PROJECT_NAME} Threads::Threads cluon-static CLI11)
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 17)

# Install executable.
install(TARGETS ${PROJECT_NAME} DESTINATION bin COMPONENT ${PROJECT_NAME})
