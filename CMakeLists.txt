# The name of our project is "VULKAN". CMakeLists files in this project can
# refer to the root source directory of the project as ${VULKAN_SOURCE_DIR} and
# to the root binary directory of the project as ${VULKAN_BINARY_DIR}.
cmake_minimum_required(VERSION 2.8.11)
project (VULKAN)
# set (CMAKE_VERBOSE_MAKEFILE 1)



# The MAJOR number of the version we're building, used in naming
# vulkan-<major>.dll (and other files).
set(MAJOR "1")


if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    add_definitions(-DVK_USE_PLATFORM_WIN32_KHR -DWIN32_LEAN_AND_MEAN)
    set(DisplayServer Win32)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Android")
    add_definitions(-DVK_USE_PLATFORM_ANDROID_KHR)
    set(DisplayServer Android)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    add_definitions(-DVK_USE_PLATFORM_XCB_KHR)
    set(DisplayServer Xcb)

#   TODO: Basic support is present for Xlib but is untested.
#         Wayland/Mir support is stubbed in but unimplemented and untested.

#   add_definitions(-DVK_USE_PLATFORM_XLIB_KHR)
#   set(DisplayServer Xlib)

#   add_definitions(-DVK_USE_PLATFORM_MIR_KHR)
#   set(DisplayServer Mir)

#   add_definitions(-DVK_USEPLATFORM_WAYLAND_KHR)
#   set(DisplayServer Wayland)

else()
    message(FATAL_ERROR "Unsupported Platform!")
endif()





set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")

# Header file for CMake settings
include_directories("${PROJECT_SOURCE_DIR}/include")

if(NOT WIN32)
    include(FindPkgConfig)
endif()

set (CMAKE_INSTALL_PREFIX "")

if (CMAKE_COMPILER_IS_GNUCC OR CMAKE_C_COMPILER_ID MATCHES "Clang")
    set(COMMON_COMPILE_FLAGS "-Wall -Wextra -Wno-unused-parameter -Wno-missing-field-initializers")
    set(COMMON_COMPILE_FLAGS "${COMMON_COMPILE_FLAGS} -fno-strict-aliasing -fno-builtin-memcmp")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 ${COMMON_COMPILE_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_COMPILE_FLAGS} -std=c++11")
    if (UNIX)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
    endif()
endif()

option(BUILD_LOADER "Build loader" ON)
option(BUILD_TESTS "Build tests" ON)
option(BUILD_LAYERS "Build layers" ON)
option(BUILD_DEMOS "Build demos" ON)
option(BUILD_VKJSON "Build vkjson" ON)

if (BUILD_TESTS)
    # Hard code our glslang path for now
    get_filename_component(GLSLANG_PREFIX ../glslang ABSOLUTE)

    if(NOT EXISTS ${GLSLANG_PREFIX})
        message(FATAL_ERROR "Necessary glslang components do not exist: " ${GLSLANG_PREFIX})
    endif()
endif()

if(NOT WIN32)
    include(GNUInstallDirs)
    add_definitions(-DSYSCONFDIR="${CMAKE_INSTALL_SYSCONFDIR}")
    add_definitions(-DDATADIR="${CMAKE_INSTALL_DATADIR}")
    if (CMAKE_INSTALL_PREFIX STREQUAL "/usr")
    elseif (CMAKE_INSTALL_PREFIX STREQUAL "")
    else()
        add_definitions(-DLOCALPREFIX="${CMAKE_INSTALL_PREFIX}")
    endif()
    if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
        set(PYTHON_CMD "python3")
    endif()
else()
    set(PYTHON_CMD "py")
endif()

# loader: Generic VULKAN ICD loader
# tests: VULKAN tests
if(BUILD_LOADER)
    add_subdirectory(loader)
endif()

if(BUILD_TESTS)
    add_subdirectory(tests)
endif()

if(BUILD_LAYERS)
    add_subdirectory(layers)
endif()

if(BUILD_DEMOS)
    add_subdirectory(demos)
endif()

if(BUILD_VKJSON)
    add_subdirectory(libs/vkjson)
endif()
