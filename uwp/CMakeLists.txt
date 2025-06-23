cmake_minimum_required(VERSION 3.14)

# Setup UWP platform
set(CMAKE_SYSTEM_NAME WindowsStore)
set(CMAKE_SYSTEM_VERSION 10.0.19041.0)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Define project
project(sk-uwp LANGUAGES CXX)
set_directory_properties(PROPERTIES VS_STARTUP_PROJECT ${PROJECT_NAME})

# Config for haxe dependency paths
set(SPAGHETTI_DIR CACHE PATH "")

add_executable(${PROJECT_NAME} WIN32
	main.cpp
)

set(PkgAssets
	Assets/LockScreenLogo.scale-200.png
	Assets/Square44x44Logo.targetsize-24_altform-unplated.png
	Assets/Square44x44Logo.scale-200.png
	Assets/Square150x150Logo.scale-200.png
	Assets/StoreLogo.png
)

set(BinDeps
	${SPAGHETTI_DIR}/Release/spaghettify.dll

	${SPAGHETTI_DIR}/mk64.o2r
	${SPAGHETTI_DIR}/spaghetti.o2r

	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/dxil.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/glfw3.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/libgallium_wgl.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/libglapi.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/libuwp.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/opengl32.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/SDL2.dll
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/bin/z-1.dll

	Package.appxmanifest
)

set(BinLibs
	${SPAGHETTI_DIR}/Release/spaghettify.lib

	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/lib/libuwp.lib
	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/lib/SDL2.lib
)
source_group("BinDeps" FILES ${BinDeps})

set_source_files_properties(${BinDeps} PROPERTIES
	VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "."
)

source_group("PkgAssets" FILES ${PkgAssets})

set_source_files_properties(${PkgAssets} PROPERTIES
	#VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "Assets"
)

# Setup data deployment
file(GLOB_RECURSE DATA_FILES
	RELATIVE ${CMAKE_SOURCE_DIR}/..
	${CMAKE_SOURCE_DIR}/../assets/*
)

foreach(ASSET ${DATA_FILES})
	set(FULL_PATH "${CMAKE_SOURCE_DIR}/../${ASSET}")

	get_filename_component(DEST_DIR ${ASSET} DIRECTORY)

	set_source_files_properties("${FULL_PATH}" PROPERTIES
		VS_COPY_TO_OUT_DIR Always
		VS_DEPLOYMENT_CONTENT TRUE
		VS_DEPLOYMENT_LOCATION "${DEST_DIR}" # preserves folder structure
	)

	target_sources(${PROJECT_NAME} PRIVATE "${FULL_PATH}")
	source_group("BinData" FILES "${FULL_PATH}")
endforeach()

target_sources(${PROJECT_NAME} PRIVATE
	${BinDeps}
	${PkgAssets}
)

target_include_directories(${PROJECT_NAME} PRIVATE
 	${SPAGHETTI_DIR}/_deps/uwplibs-src/x64/include
)

target_link_libraries(${PROJECT_NAME} PRIVATE ${BinLibs} WindowsApp.lib OneCore.lib)
set_target_properties(${PROJECT_NAME} PROPERTIES VS_WINDOWS_TARGET_PLATFORM_MIN_VERSION 10.0.19041.0)
