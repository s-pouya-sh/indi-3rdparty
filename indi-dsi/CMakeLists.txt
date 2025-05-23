cmake_minimum_required(VERSION 3.16)
PROJECT(indi-dsi CXX C)

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/")
LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake_modules/")
include(GNUInstallDirs)

option(INDI_INSTALL_UDEV_RULES "Install UDEV rules" On)
option(INDI_INSTALL_FIRMWARE "Install Firmwares" On)

IF(APPLE)
set(FIRMWARE_INSTALL_DIR "/usr/local/lib/indi/DriverSupport/dsi")
ELSE(APPLE)
set(RULES_INSTALL_DIR "/lib/udev/rules.d")
set(FIRMWARE_INSTALL_DIR "/lib/firmware")
ENDIF()
set (DSI_VERSION_MAJOR 0)
set (DSI_VERSION_MINOR 4)

find_package(CFITSIO REQUIRED)
find_package(INDI REQUIRED)
find_package(ZLIB REQUIRED)
find_package(USB1 REQUIRED)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h )
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/indi_dsi.xml.cmake ${CMAKE_CURRENT_BINARY_DIR}/indi_dsi.xml)

include_directories( ${CMAKE_CURRENT_BINARY_DIR})
include_directories( ${CMAKE_CURRENT_SOURCE_DIR})
include_directories( ${INDI_INCLUDE_DIR})
include_directories( ${CFITSIO_INCLUDE_DIR})
include_directories( ${USB1_INCLUDE_DIR})

include(CMakeCommon)

# This warning only valid for Clang above version 3.9
IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 3.8.9)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-undefined-var-template")
ENDIF ()

########### DSI ###########
set(indidsi_SRCS
   ${CMAKE_CURRENT_SOURCE_DIR}/dsi_ccd.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiDevice.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiDeviceFactory.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiPro.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiColor.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiProII.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiProIII.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiColorII.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiColorIII.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/DsiTypes.cpp
   ${CMAKE_CURRENT_SOURCE_DIR}/Util.cpp   
   )

if (APPLE)
    set(indidsi_SRCS
        ${indidsi_SRCS}
        ${CMAKE_CURRENT_SOURCE_DIR}/download_fx2.cpp)
endif()

IF (UNITY_BUILD)
    ENABLE_UNITY_BUILD(indidsi indidsi_SRCS 10 cpp)
ENDIF ()

add_executable(indi_dsi_ccd ${indidsi_SRCS})

target_link_libraries(indi_dsi_ccd ${INDI_LIBRARIES} ${CFITSIO_LIBRARIES} ${USB1_LIBRARIES} )

install(TARGETS indi_dsi_ccd RUNTIME DESTINATION bin )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/indi_dsi.xml DESTINATION ${INDI_DATA_DIR})

IF (${CMAKE_SYSTEM_NAME} MATCHES "Linux" AND INDI_INSTALL_UDEV_RULES)
install(FILES 99-meadedsi.rules DESTINATION ${RULES_INSTALL_DIR})
ENDIF()

IF (INDI_INSTALL_FIRMWARE)
    install(FILES meade-deepskyimager.hex DESTINATION ${FIRMWARE_INSTALL_DIR})
ENDIF ()
