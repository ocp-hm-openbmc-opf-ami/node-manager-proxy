cmake_minimum_required (VERSION 3.5 FATAL_ERROR)
project (node-manager-proxy CXX)
set (CMAKE_CXX_STANDARD 20)
set (CMAKE_CXX_STANDARD_REQUIRED ON)
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-rtti")

include_directories (${CMAKE_CURRENT_SOURCE_DIR})
find_package (Boost REQUIRED)
include_directories (${Boost_INCLUDE_DIRS})
add_definitions (-DBOOST_ERROR_CODE_HEADER_ONLY)
add_definitions (-DBOOST_SYSTEM_NO_DEPRECATED)
add_definitions (-DBOOST_ALL_NO_LIB)
add_definitions (-DBOOST_NO_RTTI)
add_definitions (-DBOOST_NO_TYPEID)
add_definitions (-DBOOST_ASIO_DISABLE_THREADS)

set (SRC_FILES NodeManagerProxy.cpp)

# import libsystemd
find_package (PkgConfig REQUIRED)
pkg_check_modules (SYSTEMD libsystemd REQUIRED)

# import sdbusplus
find_package (PkgConfig REQUIRED)
pkg_check_modules (SDBUSPLUSPLUS sdbusplus REQUIRED)

# import phosphor-logging
find_package (PkgConfig REQUIRED)
pkg_check_modules (LOGGING phosphor-logging REQUIRED)

add_executable (${PROJECT_NAME} ${SRC_FILES})
target_link_libraries (${PROJECT_NAME} systemd)
target_link_libraries (${PROJECT_NAME} phosphor_logging)
target_link_libraries (${PROJECT_NAME} sdbusplus -lstdc++fs)
target_link_libraries (${PROJECT_NAME} ${Boost_LIBRARIES})

link_directories (${EXTERNAL_INSTALL_LOCATION}/lib)

pkg_get_variable(SYSTEMD_SYSTEM_UNIT_DIR systemd systemdsystemunitdir)

set (SERVICE_FILES ${PROJECT_SOURCE_DIR}/node-manager-proxy.service)

install (TARGETS ${PROJECT_NAME} DESTINATION sbin)
install (FILES ${SERVICE_FILES} DESTINATION "${SYSTEMD_SYSTEM_UNIT_DIR}")
