cmake_minimum_required(VERSION 2.8)
include(cmake/defaults.cmake)

project(ShadertoyVR)

if (WIN32)
  cmake_policy(SET CMP0020 NEW)
endif (WIN32)

set(TARGET_NAME ${PROJECT_NAME})

#set(CMAKE_AUTOMOC_MOC_OPTIONS "-fCommon.h")

find_package(Threads)
list(APPEND ${PROJECT_NAME}_LIBS ${CMAKE_THREAD_LIBS_INIT})

set(QT_COMPONENTS Core Gui OpenGL Network Qml Quick Xml XmlPatterns)
find_package(Qt5 COMPONENTS ${QT_COMPONENTS} REQUIRED)
foreach(QT_MODULE ${QT_COMPONENTS})
  list(APPEND ${PROJECT_NAME}_LIBS Qt5::${QT_MODULE})
  include_directories(${Qt5${QT_MODULE}_INCLUDE_DIRS})
endforeach()

file(GLOB_RECURSE SOURCE_FILES src/*.h src/*.cpp)

include_directories(src)
include_directories(src/common)

file(GLOB_RECURSE APP_RESOURCES  res/* )
list(APPEND SOURCE_FILES ${APP_RESOURCES})

GroupSources("src")
GroupSources("res")

if(WIN32)
    add_executable(${PROJECT_NAME} WIN32 ${SOURCE_FILES})
    target_link_libraries(${PROJECT_NAME} opengl32.lib)
    copy_dlls_beside_windows_executable()
elseif(APPLE)
    add_executable(${PROJECT_NAME} MACOSX_BUNDLE ${SOURCE_FILES} )
    find_library(OpenGL OpenGL)  
else()
    add_executable(${PROJECT_NAME} ${SOURCE_FILES} )
endif()

set(PCH_FILE "${CMAKE_SOURCE_DIR}/src/common/Common.cpp")
list(REMOVE_ITEM SOURCE_FILES PCH_FILE)

add_precompiled_header("Common.h" ${PCH_FILE} SOURCE_FILES)



#
# External Dependencies
#

add_dependency_external_projects(glm)
find_package(GLM REQUIRED)
add_definitions(-DGLM_FORCE_RADIANS)
include_directories(${GLM_INCLUDE_DIRS})

add_dependency_external_projects(boostconfig)
find_package(BoostConfig REQUIRED)
include_directories(${BOOSTCONFIG_INCLUDE_DIRS})

add_dependency_external_projects(oglplus)
find_package(OGLPLUS REQUIRED)
add_definitions(-DOGLPLUS_LOW_PROFILE=1)
add_definitions(-DOGLPLUS_USE_GLEW=1)
add_definitions(-DOGLPLUS_USE_BOOST_CONFIG=1)
add_definitions(-DOGLPLUS_NO_SITE_CONFIG=1)
include_directories(${OGLPLUS_INCLUDE_DIRS})

add_dependency_external_projects(glew)
find_package(GLEW REQUIRED)
add_definitions(-DGLEW_STATIC)
list(APPEND ${PROJECT_NAME}_LIBS ${GLEW_LIBRARY})
include_directories(${GLEW_INCLUDE_DIRS})

add_dependency_external_projects(LibOVR)
find_package(LibOVR REQUIRED)
include_directories(${LIBOVR_INCLUDE_DIRS})
list(APPEND ${PROJECT_NAME}_LIBS ${LIBOVR_LIBRARIES})

target_link_libraries(${PROJECT_NAME} ${${PROJECT_NAME}_LIBS})

