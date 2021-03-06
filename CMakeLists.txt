cmake_minimum_required( VERSION 2.8 )

project( OpenARK )
set( OpenARK_VERSION_MAJOR 0 )
set( OpenARK_VERSION_MINOR 9 )
set( OpenARK_VERSION_PATCH 3 )
set( SVM_PATHS "\"svm/\", \"../svm/\", \"\"" )
set( INCLUDE_DIR "${PROJECT_SOURCE_DIR}/include" )

set( CMAKE_CXX_STACK_SIZE "10000000" )
set( CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake_modules" )

if( NOT CMAKE_BUILD_TYPE )
  set( CMAKE_BUILD_TYPE Release )
endif()

set( LIB_NAME "OpenARK" )
set( DEMO_NAME "OpenARK_hand_demo" )
set( AVATAR_DEMO_NAME "OpenARK_avatar_demo")
set( DATA_RECORDING_NAME "OpenARK_data_recording")
set( TEST_NAME "OpenARK_test" )
set( UNITY_PLUGIN_NAME "UnityPlugin" )

option( BUILD_DEMO "BUILD_DEMO" ON )
option( BUILD_AVATAR_DEMO "BUILD_AVATAR_DEMO" ON)
option( BUILD_DATA_RECORDING "BUILD_DATA_RECORDING" ON)
option( BUILD_TESTS "BUILD_TESTS" OFF )
option( BUILD_UNITY_PLUGIN "BUILD_UNITY_PLUGIN" ON )
option( USE_RSSDK2 "USE_RSSDK2" ON )
option( USE_RSSDK "USE_RSSDK" OFF )
option( USE_PMDSDK "USE_PMDSDK" OFF )

include( CheckCXXCompilerFlag )
CHECK_CXX_COMPILER_FLAG( "-std=c++11" COMPILER_SUPPORTS_CXX11 )
CHECK_CXX_COMPILER_FLAG( "-std=c++0x" COMPILER_SUPPORTS_CXX0X )

if( COMPILER_SUPPORTS_CXX11 )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11" )
elseif( COMPILER_SUPPORTS_CXX0X )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x" )
else()
    message( STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler." )
endif()

if ( CMAKE_COMPILER_IS_GNUCXX )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated -Wno-deprecated-declarations -O3 -g" )
endif ( CMAKE_COMPILER_IS_GNUCXX )

set( Boost_USE_STATIC_LIBS ON )
set( Boost_USE_STATIC ON )

find_package( PCL REQUIRED )

find_package( OpenCV REQUIRED )
if( OpenCV_FOUND )
   message( STATUS "Found OpenCV: ${OpenCV_INCLUDE_DIRS}" )
endif( OpenCV_FOUND )

set( RSSDK2_FOUND OFF )
if( USE_RSSDK2 )
  find_package( RSSDK2 )
  if( RSSDK2_FOUND )
       message( STATUS "Found RSSDK2: ${RSSDK2_INCLUDE_DIR}" )
  endif( RSSDK2_FOUND )
endif( USE_RSSDK2 )

set( RSSDK_FOUND OFF )
if( USE_RSSDK OR NOT RSSDK2_FOUND )
  find_package( RSSDK )
  if( RSSDK_FOUND )
       message( STATUS "Found RSSDK: ${RSSDK_INCLUDE_DIR}" )
  endif( RSSDK_FOUND )
endif( USE_RSSDK OR NOT RSSDK2_FOUND )

set( PMDSDK_FOUND OFF )
if( USE_PMDSDK OR ( NOT RSSDK_FOUND AND NOT RSSDK2_FOUND ) )
  find_package( PMDSDK REQUIRED )
  if( PMDSDK_FOUND )
     message( STATUS "Found PMDSDK: ${PMDSDK_INCLUDE_DIR}" )
  endif( PMDSDK_FOUND )
endif( USE_PMDSDK OR ( NOT RSSDK_FOUND AND NOT RSSDK2_FOUND ) )

include_directories(
  ${OpenCV_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
)

set(
  DEPENDENCIES
  ${OpenCV_LIBRARIES}
  ${PCL_LIBRARIES}
)

add_definitions(
  ${PCL_DEFINITIONS}
)
foreach( DEFINITION ${PCL_DEFINITIONS} )
  set( TARGET_COMPILE_FLAGS "${TARGET_COMPILE_FLAGS} ${DEFINITION}" )
endforeach()

set(
  SOURCES
  FrameObject.cpp
  Hand.cpp FramePlane.cpp
  Webcam.cpp
  DepthCamera.cpp
  RGBCamera.cpp
  StreamingAverager.cpp
  Calibration.cpp
  Util.cpp
  Visualizer.cpp
  HandClassifier.cpp
  Detector.cpp
  HandDetector.cpp
  PlaneDetector.cpp
  MockCamera.cpp
)

set(
  HEADERS
  ${INCLUDE_DIR}/FrameObject.h
  ${INCLUDE_DIR}/DetectionParams.h
  ${INCLUDE_DIR}/Hand.h
  ${INCLUDE_DIR}/FramePlane.h
  ${INCLUDE_DIR}/Webcam.h
  ${INCLUDE_DIR}/DepthCamera.h
  ${INCLUDE_DIR}/RGBCamera.h
  ${INCLUDE_DIR}/StreamingAverager.h
  ${INCLUDE_DIR}/Calibration.h
  ${INCLUDE_DIR}/Util.h
  ${INCLUDE_DIR}/Visualizer.h
  ${INCLUDE_DIR}/HandClassifier.h
  ${INCLUDE_DIR}/Detector.h
  ${INCLUDE_DIR}/HandDetector.h
  ${INCLUDE_DIR}/PlaneDetector.h
  ${INCLUDE_DIR}/MockCamera.h
  stdafx.h
)

if( NOT RSSDK2_FOUND )
  set( _RSSDK2_ "//" )
else()
  include_directories( ${RSSDK2_INCLUDE_DIR} )
  set( SOURCES ${SOURCES} RS2Camera.cpp )
  set( HEADERS ${HEADERS} ${INCLUDE_DIR}/RS2Camera.h )
  set( DEPENDENCIES ${DEPENDENCIES} ${RSSDK2_LIBRARY} )
endif( NOT RSSDK2_FOUND )

if( NOT RSSDK_FOUND )
  set( _RSSDK_ "//" )
else()
  include_directories( ${RSSDK_INCLUDE_DIR} )
  set( SOURCES ${SOURCES} SR300Camera.cpp Converter.cpp )
  set( HEADERS ${HEADERS} ${INCLUDE_DIR}/SR300Camera.h ${INCLUDE_DIR}/Converter.h )
endif( NOT RSSDK_FOUND )

if( NOT PMDSDK_FOUND )
  set( _PMDSDK_ "//" )
else()
  include_directories( ${PMDSDK_INCLUDE_DIR} )
  set( SOURCES ${SOURCES} PMDCamera.cpp )
  set( HEADERS ${HEADERS} ${INCLUDE_DIR}/PMDCamera.h )
  set( DEPENDENCIES ${DEPENDENCIES} ${PMDSDK_LIBRARIES} )
endif( NOT PMDSDK_FOUND )

if ( NOT CMAKE_BUILD_TYPE STREQUAL "Debug" )
  set( _DEBUG_ "//" )
endif ( NOT CMAKE_BUILD_TYPE STREQUAL "Debug" )

configure_file( Version.h.in ${INCLUDE_DIR}/Version.h )
set( HEADERS ${HEADERS} ${INCLUDE_DIR}/Version.h )

# MSVC special options
if ( MSVC )
    # MSVC precompiled header
    set( PRECOMPILED_BINARY "$(IntDir)/stdafx.pch" )
    set( PRECOMPILED_HEADER "stdafx.h" )
    set( PRECOMPILED_SOURCE "stdafx.cpp" )

    set_source_files_properties( ${PRECOMPILED_SOURCE}
                                PROPERTIES COMPILE_FLAGS "/Yc\"${PRECOMPILED_HEADER}\" /Fp\"${PRECOMPILED_BINARY}\""
                                           OBJECT_OUTPUTS "${PRECOMPILED_HEADER}" )
    set_source_files_properties( ${SOURCES}
                                PROPERTIES COMPILE_FLAGS "/Yu\"${PRECOMPILED_HEADER}\" /FI\"${PRECOMPILED_HEADER}\" /Fp\"${PRECOMPILED_BINARY}\""
                                           OBJECT_DEPENDS "${PRECOMPILED_HEADER}" )
    set( SOURCES ${SOURCES} ${PRECOMPILED_SOURCE} )
    add_definitions( -D_CRT_SECURE_NO_WARNINGS )
    set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc /Ot" )
    set( CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} /NODEFAULTLIB:LIBCMT" )
    set( CMAKE_SHARED_LINKER_FLAGS  "${CMAKE_SHARED_LINKER_FLAGS} /NODEFAULTLIB:LIBCMT" )
    set( CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD" )
endif ( MSVC )

add_library( ${LIB_NAME} STATIC "${INCLUDE_DIR}/Core.h" ${SOURCES} ${HEADERS} )
set_target_properties( ${LIB_NAME} PROPERTIES OUTPUT_NAME
        "openark_${OpenARK_VERSION_MAJOR}_${OpenARK_VERSION_MINOR}_${OpenARK_VERSION_PATCH}" )

set_property( TARGET ${LIB_NAME} PROPERTY DBG_POSTFIX d )
target_link_libraries( ${LIB_NAME} ${DEPENDENCIES} )
target_include_directories( ${LIB_NAME} PRIVATE ${INCLUDE_DIR} )

if ( MSVC )
    set_target_properties( ${LIB_NAME} PROPERTIES COMPILE_FLAGS "/MD" )
endif()

if( ${BUILD_DEMO} )
    add_executable( ${DEMO_NAME} HandDemo.cpp )
    target_include_directories( ${DEMO_NAME} PRIVATE ${INCLUDE_DIR} )
    target_link_libraries( ${DEMO_NAME} ${DEPENDENCIES} ${LIB_NAME} )
    set_target_properties( ${DEMO_NAME} PROPERTIES OUTPUT_NAME ${LIB_NAME} )
    set_target_properties( ${DEMO_NAME} PROPERTIES COMPILE_FLAGS ${TARGET_COMPILE_FLAGS} )
endif( ${BUILD_DEMO} )

if( ${BUILD_AVATAR_DEMO} )
    add_executable( ${AVATAR_DEMO_NAME} AvatarDemo.cpp )
    target_include_directories( ${AVATAR_DEMO_NAME} PRIVATE ${INCLUDE_DIR} )
    target_link_libraries( ${AVATAR_DEMO_NAME} ${DEPENDENCIES} ${LIB_NAME} )
    set_target_properties( ${AVATAR_DEMO_NAME} PROPERTIES OUTPUT_NAME ${LIB_NAME} )
    set_target_properties( ${AVATAR_DEMO_NAME} PROPERTIES COMPILE_FLAGS ${TARGET_COMPILE_FLAGS} )
endif( ${BUILD_AVATAR_DEMO} )

if( ${BUILD_DATA_RECORDING} )
    add_executable( ${DATA_RECORDING_NAME} DataRecording.cpp )
    target_include_directories( ${DATA_RECORDING_NAME} PRIVATE ${INCLUDE_DIR} )
    target_link_libraries( ${DATA_RECORDING_NAME} ${DEPENDENCIES} ${LIB_NAME} )
    set_target_properties( ${DATA_RECORDING_NAME} PROPERTIES OUTPUT_NAME ${LIB_NAME} )
    set_target_properties( ${DATA_RECORDING_NAME} PROPERTIES COMPILE_FLAGS ${TARGET_COMPILE_FLAGS} )
endif( ${BUILD_DATA_RECORDING} )

# Unity plugin currently only supports Windows
if( ${BUILD_UNITY_PLUGIN} AND MSVC )
    add_library( ${UNITY_PLUGIN_NAME} SHARED "unity/native/UnityInterface.cpp" "unity/native/UnityInterface.h" "unity/README.md" )
    set( HEADERS ${HEADERS} "unity/native/UnityInterface.h" )
    target_link_libraries( ${UNITY_PLUGIN_NAME} ${DEPENDENCIES} ${LIB_NAME} )
    if ( MSVC )
        set_target_properties( ${UNITY_PLUGIN_NAME} PROPERTIES COMPILE_FLAGS "/MD" )
    endif ( MSVC )
    set_property( TARGET ${UNITY_PLUGIN_NAME} PROPERTY DBG_POSTFIX d )
    target_include_directories( ${UNITY_PLUGIN_NAME} PRIVATE ${INCLUDE_DIR} "unity/native" )
    set_target_properties( ${UNITY_PLUGIN_NAME} PROPERTIES OUTPUT_NAME "openark_unity_${OpenARK_VERSION_MAJOR}_${OpenARK_VERSION_MINOR}_${OpenARK_VERSION_PATCH}_native" )
endif( ${BUILD_UNITY_PLUGIN} AND MSVC )

if( ${BUILD_TESTS} )
  if( NOT EXISTS ${PROJECT_SOURCE_DIR}/OpenARK_test )
    execute_process(
      COMMAND git clone https://github.com/augcog/OpenARK_test.git
      WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} )
  endif( NOT EXISTS ${PROJECT_SOURCE_DIR}/OpenARK_test )

  add_executable( ${TEST_NAME} "OpenARK_test/test.cpp" "OpenARK_test/TestCamera.cpp" "OpenARK_test/TestCamera.h" )
  set( HEADERS ${HEADERS} "OpenARK_test/TestCamera.h" )

  target_link_libraries( ${TEST_NAME} ${DEPENDENCIES} ${LIB_NAME} )
  set_target_properties( ${TEST_NAME} PROPERTIES
    COMPILE_FLAGS ${TARGET_COMPILE_FLAGS} )

  target_include_directories( ${TEST_NAME} PRIVATE ${INCLUDE_DIR} )
endif( ${BUILD_TESTS} )

# Create source group for headers
source_group( "Header Files" FILES ${HEADERS} )