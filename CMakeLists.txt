cmake_minimum_required(VERSION 3.20 FATAL_ERROR)
cmake_policy(VERSION 3.20)

project(qpsd)

# load paths from the user file if exists
if (EXISTS ${CMAKE_SOURCE_DIR}/CMakeUserPaths.cmake)
  include(${CMAKE_SOURCE_DIR}/CMakeUserPaths.cmake)
		
	message(STATUS "using user paths...")
elseif(MSVC)
	message(WARNING "Could not find CMakeUserPaths.cmake - please use this file to specify your install directories (see CMakeUserPathsGit.cmake)")
endif()

# locate qmake
if(NOT QT_QMAKE_EXECUTABLE)
  find_program(QT_QMAKE_EXECUTABLE NAMES "qmake" "qmake-qt5" "qmake.exe")
endif()
if(NOT QT_QMAKE_EXECUTABLE)
  message(FATAL_ERROR "you have to set the path to the Qt5 qmake executable in CMakeUserPaths.cmake")
else()
  message(STATUS "QMake found: ${QT_QMAKE_EXECUTABLE}")
endif()

get_filename_component(QT_QMAKE_PATH ${QT_QMAKE_EXECUTABLE} PATH)

set(QT_ROOT ${QT_QMAKE_PATH}/)
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} ${QT_QMAKE_PATH}/../lib/cmake/Qt${QT_VERSION_MAJOR})


find_package(QT NAMES Qt6 Qt5 COMPONENTS Core REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Core Gui REQUIRED)
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

file(GLOB LIBQPSD_SOURCES "*.cpp")
file(GLOB LIBQPSD_HEADERS "*.h")

SET(CMAKE_DEBUG_POSTFIX "d")

if(APPLE)
    add_library(${PROJECT_NAME} SHARED ${LIBQPSD_SOURCES} ${LIBQPSD_HEADERS})
else()
    add_library(${PROJECT_NAME} MODULE ${LIBQPSD_SOURCES} ${LIBQPSD_HEADERS})
endif()

target_link_libraries(${PROJECT_NAME}
    Qt${QT_VERSION_MAJOR}::Core
    Qt${QT_VERSION_MAJOR}::Gui
)

set_target_properties(${PROJECT_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_BINARY_DIR}/libs)
set_target_properties(${PROJECT_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_BINARY_DIR}/libs)
set_target_properties(${PROJECT_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY_REALLYRELEASE ${CMAKE_CURRENT_BINARY_DIR}/libs)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/imageformats")

# install to qt plugins dir
execute_process(
  COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_INSTALL_PLUGINS
  OUTPUT_VARIABLE QT_PLUGINS_DIR
  OUTPUT_STRIP_TRAILING_WHITESPACE)

# Prefix with DESTDIR if available to allow packaging
if (DEFINED ENV{DESTDIR})
  set(plugins_dir "$ENV{DESTDIR}/plugins")
else ()
  set(plugins_dir "${QT_PLUGINS_DIR}")
endif ()

set(INSTALL_PATH "${plugins_dir}/imageformats")

cmake_path(ABSOLUTE_PATH INSTALL_PATH NORMALIZE)

install(
  TARGETS ${PROJECT_NAME}
  LIBRARY DESTINATION ${INSTALL_PATH})

if (QT_PLUGINS_DIR)
  # status
  message(STATUS "")
  message(STATUS "-----------------------------------------------------------------------------")
  message(STATUS " ${PROJECT_NAME} configured")
  message(STATUS " <https://github.com/Code-ReaQtor/libqpsd>")
  message(STATUS " it will be installed to ${INSTALL_PATH}")
  message(STATUS "-----------------------------------------------------------------------------")
else ()
  message(FATAL_ERROR "Qt5 plugin directory cannot be detected.")
endif ()
