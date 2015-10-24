project(ssl-vision)
cmake_minimum_required(VERSION 2.8.6)

# defines
add_definitions(-D UNIX)
add_definitions(-D LINUX)
add_definitions(-D _GNU_SOURCE)

message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")

set(arch ${CMAKE_SYSTEM_PROCESSOR})
message(STATUS "Arch: ${arch}")

if(${arch} MATCHES "x86_64")
	# set compiler flags here for x86_64
	set(CMAKE_CXX_FLAGS "")
elseif(${arch} MATCHES "x86")
	# set compiler flags here for x86
	set(CMAKE_CXX_FLAGS "")
endif()

add_definitions(-Wall -Wextra)
set(CMAKE_CXX_FLAGS_DEBUG   "-g -Wl,--no-as-needed")
set(CMAKE_CXX_FLAGS_RELEASE "-DNDEBUG -O3 -march=native -Wl,--no-as-needed")
set(CMAKE_EXE_LINKER_FLAGS  "-s")

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

# Find third-party libraries
find_package(OpenGL REQUIRED)
find_package(Qt4 4.8.0 COMPONENTS QtCore QtGui QtOpenGL QtNetwork REQUIRED)
find_package(Threads REQUIRED)

find_package(Protobuf REQUIRED)
if(EXISTS ${PROTOBUF_PROTOC_EXECUTABLE})
	message(STATUS "Found PROTOBUF Compiler: ${PROTOBUF_PROTOC_EXECUTABLE}")
else()
	message(FATAL_ERROR "Could not find PROTOBUF Compiler")
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(Eigen2 REQUIRED eigen2)
pkg_check_modules(DC1394 REQUIRED libdc1394-2)

include_directories(
	${Eigen2_INCLUDE_DIRS}
	${PROTOBUF_INCLUDE_DIRS}
	${PROJECT_BINARY_DIR}
	${PROJECT_SOURCE_DIR}/src/app
	${PROJECT_SOURCE_DIR}/src/app/gui
	${PROJECT_SOURCE_DIR}/src/app/plugins
	${PROJECT_SOURCE_DIR}/src/app/stacks
)

set(CMAKE_AUTOMOC ON)
include(${QT_USE_FILE})

include(src/shared/CMakeLists.txt.inc)

set(SRCS
	src/app/capture_thread.cpp
	src/app/framedata.cpp
	src/app/main.cpp

	src/app/gui/cameracalibwidget.cpp
	src/app/gui/colorpicker.cpp
	src/app/gui/glLUTwidget.cpp
	src/app/gui/glwidget.cpp
	src/app/gui/lutwidget.cpp
	src/app/gui/mainwindow.cpp
	src/app/gui/realtimedisplaywidget.cpp
	src/app/gui/renderoptions.cpp
	src/app/gui/videowidget.cpp
	src/app/gui/jog_dial.cpp

	src/app/plugins/plugin_cameracalib.cpp
	src/app/plugins/plugin_colorcalib.cpp
	src/app/plugins/plugin_colorthreshold.cpp
	src/app/plugins/plugin_detect_balls.cpp
	src/app/plugins/plugin_detect_robots.cpp
	src/app/plugins/plugin_find_blobs.cpp
	src/app/plugins/plugin_publishgeometry.cpp
	src/app/plugins/plugin_legacypublishgeometry.cpp
	src/app/plugins/plugin_runlength_encode.cpp
	src/app/plugins/plugin_sslnetworkoutput.cpp
	src/app/plugins/plugin_legacysslnetworkoutput.cpp
	src/app/plugins/plugin_visualize.cpp
	src/app/plugins/plugin_dvr.cpp
	src/app/plugins/visionplugin.cpp

	src/app/stacks/multistack_robocup_ssl.cpp
	src/app/stacks/multivisionstack.cpp
	src/app/stacks/stack_robocup_ssl.cpp
	src/app/stacks/visionstack.cpp
)

qt4_wrap_ui(UI_SRCS
	src/app/gui/mainwindow.ui
	src/app/gui/videowidget.ui
)

qt4_add_resources(RC_SRCS 
	src/app/gui/icons/icons_gui.qrc
	${SHARED_RC}
)

qt4_add_resources(SHARED_RC_SRCS 
	${SHARED_RC}
)

## build the common code
add_library(sslvision ${SHARED_RC_SRCS} ${CC_PROTO} ${SHARED_SRCS})

set(libs
	${CMAKE_THREAD_LIBS_INIT}
	${DC1394_LIBRARIES}
	${OPENGL_gl_LIBRARY}
	${OPENGL_glu_LIBRARY}
	${PROTOBUF_LIBRARIES}
	${QT_LIBRARIES}
	sslvision
)

## build the main app
set(target vision)
add_executable(${target} ${UI_SRCS} ${RC_SRCS} ${SRCS})
target_link_libraries(${target} ${libs})

##build non graphical client
set(client client)
add_executable(${client} src/client/main.cpp )
target_link_libraries(${client} ${libs})

## build logging client
set(lclient logClient)
add_executable(${lclient}
	src/logClient/main.cpp 
	src/logClient/CentralWindow.cpp
	src/logClient/ClientThreading.cpp
	src/logClient/GraphicsPrimitives.cpp
	src/logClient/LogControl.cpp
)
target_link_libraries(${lclient} ${libs})

## build graphical client
set(gclient graphicalClient)
add_executable(${gclient}
	src/graphicalClient/main.cpp
	src/graphicalClient/soccerview.cpp
	src/graphicalClient/gltext.cpp
)
target_link_libraries(${gclient} ${libs})

