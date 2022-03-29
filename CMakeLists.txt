cmake_minimum_required(VERSION 3.10.0)
project(smoothfeatureline)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

find_package(LIBIGL QUIET)

if (NOT LIBIGL_FOUND)
   message(FATAL_ERROR "libigl not found --- You can download it using: \n git clone --recursive https://github.com/libigl/libigl.git ${PROJECT_SOURCE_DIR}/../libigl")
endif()

# Compilation flags: adapt to your needs 
if(MSVC)
  # Enable parallel compilation
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP /bigobj") 
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR} )
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR} )
else()
  # Libigl requires a modern C++ compiler that supports c++11
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11") 
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "." )
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")
endif()

# libigl options: choose between header only and compiled static library
# Header-only is preferred for small projects. For larger projects the static build
# considerably reduces the compilation times
option(LIBIGL_USE_STATIC_LIBRARY "Use LibIGL as static library" OFF)

# add a customizable menu bar
option(LIBIGL_WITH_NANOGUI     "Use Nanogui menu"   ON)

# libigl options: choose your dependencies (by default everything is OFF except opengl) 
#option(LIBIGL_EMBREE           "Build target igl::embree"           ON)
option(LIBIGL_GLFW             "Build target igl::glfw"             ON)
option(LIBIGL_IMGUI            "Build target igl::imgui"            ON)
option(LIBIGL_OPENGL           "Build target igl::opengl"           ON)
#option(LIBIGL_PNG              "Build target igl::png"              ON)
#option(LIBIGL_PREDICATES       "Build target igl::predicates"       ON)
#option(LIBIGL_XML              "Build target igl::xml"              ON)

if(LIBIGL_WITH_CGAL) # Do not remove or move this block, the cgal build system fails without it
  find_package(CGAL REQUIRED)
  set(CGAL_DONT_OVERRIDE_CMAKE_FLAGS TRUE CACHE BOOL "CGAL's CMAKE Setup is super annoying ")
  include(${CGAL_USE_FILE})
endif()

option(FETCHCONTENT_SOURCE_DIR_LIBIGL ON)

# Adding libigl: choose the path to your local copy libigl 
# This is going to compile everything you requested 
#message(FATAL_ERROR "${PROJECT_SOURCE_DIR}/../libigl/cmake")
#add_definitions(-Wno-unused-variable) # Embree in LibIGL currently emits unused var warnings
#add_subdirectory("${LIBIGL_INCLUDE_DIR}/../shared/cmake" "libigl")
add_subdirectory(../common-deps/deps/libigl deps/libigl)

remove_definitions(-Wno-unused-variable) # We want unused var warnings for our code...
add_definitions(-Wall)          # ... and everything else
add_definitions(-Wsign-compare) # (Clang doesn't care about comparison signedness by default...)

# libigl information 
message("libigl includes: ${LIBIGL_INCLUDE_DIRS}")
message("libigl libraries: ${LIBIGL_LIBRARIES}")
message("libigl extra sources: ${LIBIGL_EXTRA_SOURCES}")
message("libigl extra libraries: ${LIBIGL_EXTRA_LIBRARIES}")
message("libigl definitions: ${LIBIGL_DEFINITIONS}")

# Prepare the build environment
include_directories(${LIBIGL_INCLUDE_DIRS})
add_definitions(${LIBIGL_DEFINITIONS})

# Add your project files
# Note: we need some stubs for building to succeed on TravisCI.
if (TRAVISCI_BUILD)
    FILE(GLOB SOURCES src/*.cpp src_travisci/*.cpp)
else (TRAVISCI_BUILD)
    FILE(GLOB SOURCES src/*.cpp)
endif (TRAVISCI_BUILD)

find_package (Eigen3 3.3 REQUIRED NO_MODULE)
include_directories(${EIGEN_INCLUDE_DIR})

add_executable(${PROJECT_NAME}_bin ${SOURCES} ${LIBIGL_EXTRA_SOURCES})
target_link_libraries(${PROJECT_NAME}_bin ${LIBIGL_LIBRARIES} ${LIBIGL_EXTRA_LIBRARIES} Eigen3::Eigen igl::core igl::opengl igl::glfw igl::imgui)
target_include_directories(${PROJECT_NAME}_bin SYSTEM PRIVATE ${LIBIGL_INCLUDE_DIRS})

# Add libIGL in header-only mode.
#target_include_directories(${PROJECT_NAME}_bin PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/../common-deps/deps/libigl/include")
