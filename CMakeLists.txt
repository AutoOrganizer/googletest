# Note: CMake support is community-based. The maintainers do not use CMake
# internally.

cmake_minimum_required(VERSION 3.16)

if (DEFINED TARGET_ARCH AND "${TARGET_ARCH}" STREQUAL "x86" AND NOT "${CMAKE_CXX_PLATFORM_ID}" STREQUAL "Windows")
    string(APPEND CMAKE_C_FLAGS_INIT " -m32")
    string(APPEND CMAKE_CXX_FLAGS_INIT " -m32")
endif()

project(googletest-distribution)
set(GOOGLETEST_VERSION 1.16.0)

if(NOT CYGWIN AND NOT MSYS AND NOT ${CMAKE_SYSTEM_NAME} STREQUAL QNX)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

enable_testing()

include(CMakeDependentOption)
include(GNUInstallDirs)

# Note that googlemock target already builds googletest.
option(BUILD_GMOCK "Builds the googlemock subproject" ON)
option(INSTALL_GTEST "Enable installation of googletest. (Projects embedding googletest may want to turn this OFF.)" ON)
option(GTEST_HAS_ABSL "Use Abseil and RE2. Requires Abseil and RE2 to be separately added to the build." OFF)

if(GTEST_HAS_ABSL)
  if(NOT TARGET absl::base)
    find_package(absl REQUIRED)
  endif()
  if(NOT TARGET re2::re2)
    find_package(re2 REQUIRED)
  endif()
endif()

set(ARCH_DETECT [=[
string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" tempResult)
if(tempResult MATCHES "^(x86_64|amd64|i[3-6]86|x86)$")
    if(CMAKE_SIZEOF_VOID_P EQUAL 4)
        set(DETECTED_ARCH x86)
    elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(DETECTED_ARCH x64)
    else()
        message(FATAL_ERROR "Unsupported x86 pointer size: ${CMAKE_SIZEOF_VOID_P}")
    endif()
elseif(tempResult MATCHES "^(aarch64|arm64)$")
    set(DETECTED_ARCH arm64)
else()
    message(FATAL_ERROR "Unsupported target processor: ${CMAKE_SYSTEM_PROCESSOR}")
endif()
]=])
cmake_language(EVAL CODE "${ARCH_DETECT}")

if(BUILD_GMOCK)
  add_subdirectory( googlemock )
else()
  add_subdirectory( googletest )
endif()

if("${DETECTED_ARCH}" STREQUAL "x64")
    set_target_properties(gtest gmock gtest_main gmock_main PROPERTIES 
        DEBUG_POSTFIX d-x64
        RELEASE_POSTFIX -x64
        MINSIZEREL_POSTFIX -x64
        RELWITHDEBINFO_POSTFIX -x64
    )
else()
    set_target_properties(gtest gmock gtest_main gmock_main PROPERTIES 
        DEBUG_POSTFIX d
    )
endif()