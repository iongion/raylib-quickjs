cmake_minimum_required(VERSION 3.1)
set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64" CACHE STRING "" FORCE)

# Workaround for windows to prevent static linking of pthread
IF (WIN32)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -static")
ENDIF()

project(rayjs)

message("=== Configure raylib ===")
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/raylib EXCLUDE_FROM_ALL)

message("=== Configure QuickJS ===")
# add quickjs
set(quickjs_version 2021-03-27)
set(quickjs_sources_root ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/quickjs)
set(quickjs_sources
	${quickjs_sources_root}/quickjs.h
	#${quickjs_sources_root}/quickjs-libc.h
	${quickjs_sources_root}/quickjs.c
	${quickjs_sources_root}/libregexp.c
	${quickjs_sources_root}/libunicode.c
	${quickjs_sources_root}/libbf.c
	${quickjs_sources_root}/cutils.c
	#${quickjs_sources_root}/quickjs-libc.c
)
add_library(quickjs STATIC
	${quickjs_sources}
)
target_compile_definitions(quickjs
	PRIVATE
		CONFIG_VERSION="${quickjs_version}"
)
target_include_directories(quickjs
	PRIVATE
		${quickjs_sources_root}
)

add_library(lightmapper INTERFACE)
target_include_directories(lightmapper INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/lightmapper)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
file(GLOB files src/*.c)
#add_executable(${CMAKE_PROJECT_NAME} MACOSX_BUNDLE ${files})
add_executable(${CMAKE_PROJECT_NAME} ${files})
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE include)
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE src)
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE thirdparty/raygui/src)
target_link_libraries(${CMAKE_PROJECT_NAME} quickjs raylib lightmapper)
