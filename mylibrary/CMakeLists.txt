cmake_minimum_required(VERSION 3.4.1)
add_library(native-lib SHARED myfile.cpp)
#find_library(log-lib log)
target_link_libraries(native-lib)

if(DEFINED RAPTOR_CAGE_ENABLED)
  raptor_cage()
endif()

# Android NDK
get_filename_component(TEST_ANDROID_NDKEXPECTED_PATH
    "${CMAKE_CURRENT_LIST_DIR}/../.." ABSOLUTE)
if(NOT ANDROID_NDK)
  set(ANDROID_NDK "${TEST_ANDROID_NDKEXPECTED_PATH}")
else()
  # Allow the user to specify their own NDK path, but emit a warning. This is an
  # uncommon use case, but helpful if users want to use a bleeding edge
  # toolchain file with a stable NDK.
  # https://github.com/android-ndk/ndk/issues/473
  get_filename_component(ANDROID_NDK "${ANDROID_NDK}" ABSOLUTE)
endif()
unset(TEST_ANDROID_NDKEXPECTED_PATH)
file(TO_CMAKE_PATH "${ANDROID_NDK}" ANDROID_NDK)

# Android NDK revision
# Possible formats:
# * r16, build 1234: 16.0.1234
# * r16b, build 1234: 16.1.1234
# * r16 beta 1, build 1234: 16.0.1234-beta1
#
# Canary builds are not specially marked.
file(READ "${ANDROID_NDK}/source.properties" TEST_ANDROID_NDKSOURCE_PROPERTIES)

set(TEST_ANDROID_NDKREVISION_REGEX
  "^Pkg\\.Desc = Android NDK\nPkg\\.Revision = ([0-9]+)\\.([0-9]+)\\.([0-9]+)(-beta([0-9]+))?")
if(NOT TEST_ANDROID_NDKSOURCE_PROPERTIES MATCHES "${TEST_ANDROID_NDKREVISION_REGEX}")
  message(SEND_ERROR "Failed to parse Android NDK revision: ${ANDROID_NDK}/source.properties.\n${TEST_ANDROID_NDKSOURCE_PROPERTIES}")
endif()

set(TEST_ANDROID_NDKMAJOR "${CMAKE_MATCH_1}")
set(TEST_ANDROID_NDKMINOR "${CMAKE_MATCH_2}")
set(TEST_ANDROID_NDKBUILD "${CMAKE_MATCH_3}")
set(TEST_ANDROID_NDKBETA "${CMAKE_MATCH_5}")
if(TEST_ANDROID_NDKBETA STREQUAL "")
  set(TEST_ANDROID_NDKBETA "0")
endif()
set(TEST_ANDROID_NDKREVISION
  "${TEST_ANDROID_NDKMAJOR}.${TEST_ANDROID_NDKMINOR}.${TEST_ANDROID_NDKBUILD}${CMAKE_MATCH_4}")

set(CAP variables-${TEST_ANDROID_NDKREVISION}-${ANDROID_ABI}-${ANDROID_PLATFORM}-${CMAKE_BUILD_TYPE}-${CMAKE_ANDROID_ARCH_ABI}.txt)
#file(APPEND ${CAP} "ANDROID_ABI=${ANDROID_ABI}\n")
#file(APPEND ${CAP} "ANDROID_PLATFORM=${ANDROID_PLATFORM}\n")
#file(APPEND ${CAP} "CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}\n")
#file(APPEND ${CAP} "CMAKE_ANDROID_ARCH_ABI=${CMAKE_ANDROID_ARCH_ABI}\n")
#file(APPEND ${CAP} "CMAKE_SYSTEM_VERSION=${CMAKE_SYSTEM_VERSION}\n")
get_cmake_property(_variableNames VARIABLES)
list (SORT _variableNames)
foreach (_variableName ${_variableNames})
    file(APPEND ${CAP} "set ${_variableName}=${${_variableName}}\n")
endforeach()