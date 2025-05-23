########### Celestron AUX INDI driver ##############
cmake_minimum_required(VERSION 3.16)
PROJECT(indi-celestronaux C CXX)


include(GNUInstallDirs)

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/")
LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake_modules/")

find_package(INDI REQUIRED)
find_package(Nova REQUIRED)
find_package(ZLIB REQUIRED)
find_package(GSL REQUIRED)

set(CAUX_VERSION_MAJOR 1)
set(CAUX_VERSION_MINOR 4)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h )
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/indi_celestronaux.xml.cmake ${CMAKE_CURRENT_BINARY_DIR}/indi_celestronaux.xml )

include_directories( ${CMAKE_CURRENT_BINARY_DIR})
include_directories( ${CMAKE_CURRENT_SOURCE_DIR})
include_directories( ${INDI_INCLUDE_DIR})
include_directories( ${NOVA_INCLUDE_DIR})
include_directories( ${EV_INCLUDE_DIR})

include(CMakeCommon)

add_executable(indi_celestron_aux auxproto.cpp celestronaux.cpp adaptive_tuner.cpp)
target_link_libraries(indi_celestron_aux ${INDI_LIBRARIES} ${NOVA_LIBRARIES} ${GSL_LIBRARIES})
install(TARGETS indi_celestron_aux RUNTIME DESTINATION bin)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/indi_celestronaux.xml DESTINATION ${INDI_DATA_DIR})
