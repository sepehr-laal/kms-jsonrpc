cmake_minimum_required(VERSION 2.8)

project("kms-jsonrpc")
message(STATUS "Running CMake for project: ${PROJECT_NAME}")

message("If KurentoHelpers is not found, you need to install 'kms-cmake-utils' from the Kurento repository")
find_package(KurentoHelpers REQUIRED)

# Set GCC flags common to all projects
# Note - default flags added by dpkg-buildflags for C and C++:
# -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2
include(DpkgBuildFlags)
get_dpkg_buildflags_c(DPKG_BUILDFLAGS_C)
get_dpkg_buildflags_cxx(DPKG_BUILDFLAGS_CXX)
set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   ${DPKG_BUILDFLAGS_C}   -std=c11   -Wall -Werror")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${DPKG_BUILDFLAGS_CXX} -std=c++11 -Wall -Werror")

include(KurentoGitHelpers)
get_git_version(PROJECT_VERSION 1.1.3-dev)
install_git_hook(pre-commit ${CMAKE_CURRENT_SOURCE_DIR}/hooks/pre-commit.hook)

find_package(PkgConfig)
pkg_check_modules(JSONCPP REQUIRED kmsjsoncpp>=0.6.0)

find_package(Boost REQUIRED COMPONENTS unit_test_framework)

include(GNUInstallDirs)

set(CUSTOM_PREFIX "kurento")
set(INCLUDE_PREFIX "${CMAKE_INSTALL_INCLUDEDIR}/${CUSTOM_PREFIX}/jsonrpc")

if(NOT DEFINED SYSCONF_INSTALL_DIR)
  set(SYSCONF_INSTALL_DIR etc CACHE PATH
      "Install directory for system-wide configuration files")
endif(NOT DEFINED SYSCONF_INSTALL_DIR)

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix "\${prefix}")
set(libdir "\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}")
set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}/${CUSTOM_PREFIX}")
set(requires "kmsjsoncpp")

configure_file(libjsonrpc.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libjsonrpc.pc @ONLY)
configure_file(FindKmsJsonRpc.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/FindKmsJsonRpc.cmake @ONLY)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/libjsonrpc.pc
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
  )

set(CMAKE_MODULES_INSTALL_DIR
  ${CMAKE_INSTALL_DATAROOTDIR}/cmake-${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}/Modules
  CACHE STRING
  "Destination (relative to CMAKE_INSTALL_PREFIX) for cmake modules files"
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/FindKmsJsonRpc.cmake
  DESTINATION ${CMAKE_MODULES_INSTALL_DIR}
)

enable_testing()

add_subdirectory(src)
add_subdirectory(tests)
