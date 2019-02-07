### 1 - Required CMake Version
cmake_minimum_required(VERSION 2.8.3)
### 2- Package Name
project(picotts)

IF(NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    add_definitions(-DCOMPILEDWITHC11)
    message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    add_definitions(-DCOMPILEDWITHC0X)
    message(STATUS "Using flag -std=c++0x.")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3")

### 3 - Find other CMake/Catkin packages needed for build
SET(DEPS  roscpp  roslib  std_msgs)
find_package(catkin REQUIRED COMPONENTS ${DEPS})
# find all the libraries needed by libnotify
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/CMake/Modules")
find_package(GDK-PixBuf REQUIRED)
find_package(Glib REQUIRED)
find_package(GObject REQUIRED)
find_package(Libnotify REQUIRED)
# nothing to do in this package
### 4 - Message/Service/Action Generators (add_message_files(), add_service_files(), add_action_files())
# nothing to do in this package
### 5 - Invoke message/service/action generation (generate_messages())
# nothing to do in this package
### 6 - Specify package build info export (catkin_package())
catkin_package(
  INCLUDE_DIRS   include
  CATKIN_DEPENDS ${DEPS}
  DEPENDS
)
### 7 - Libraries/Executables to build (add_library()/add_executable()/target_link_libraries())
SET(PKG_INC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include/picotts)
include_directories(include
  ${catkin_INCLUDE_DIRS}
  ${GDK-PixBuf_INCLUDE_DIRS}
  ${Glib_INCLUDE_DIRS}
  ${GOBJECT_INCLUDE_DIR}
  ${LIBNOTIFY_INCLUDE_DIR}
)
add_executable       (picotts.exe  src/picotts.cpp
  ${PKG_INC_DIR}/cached_files_map.h
  ${PKG_INC_DIR}/utils.h)
target_link_libraries(picotts.exe  ${catkin_LIBRARIES})

add_executable       (tts2notify.exe  src/tts2notify.cpp)
target_link_libraries(tts2notify.exe  ${catkin_LIBRARIES}
  ${GDK-PixBuf_LIBRARIES}
  ${Glib_LIBRARIES}
  ${GOBJECT_LIBRARIES}
  ${LIBNOTIFY_LIBRARIES}
)

add_executable       (test_picotts.exe  src/test_picotts.cpp
  ${PKG_INC_DIR}/picotts_publisher.h)
target_link_libraries(test_picotts.exe  ${catkin_LIBRARIES})
### 8 - Tests to build (catkin_add_gtest())
# nothing to do in this package
### 9 - Install rules (install())
install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
   FILES_MATCHING PATTERN "*.h"
   PATTERN ".svn" EXCLUDE
)
