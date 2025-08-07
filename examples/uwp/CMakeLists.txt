cmake_minimum_required(VERSION 3.14)

# Setup UWP platform
set(CMAKE_SYSTEM_NAME WindowsStore)
set(CMAKE_SYSTEM_VERSION 10.0.19041.0)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type (Debug, Release, RelWithDebInfo, MinSizeRel)" FORCE)
endif()

# Define project
project(plume-uwp LANGUAGES CXX)
set_directory_properties(PROPERTIES VS_STARTUP_PROJECT ${PROJECT_NAME})

# Config for haxe dependency paths
set(PLUME_DIR "" CACHE PATH "")
if(NOT PLUME_DIR)
  message(FATAL_ERROR "PLUME_DIR must be set to a valid path pointing to the static lib build")
endif()

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

set(AgilityBins
	${PLUME_DIR}/_deps/uwpdep-src/x64/bin/D3D12/D3D12Core.dll
	${PLUME_DIR}/_deps/uwpdep-src/x64/bin/D3D12/d3d12SDKLayers.dll
)

set(BinDeps
	${AgilityBins}
	${PLUME_DIR}/_deps/dxc-src/bin/x64/dxcompiler.dll
	${PLUME_DIR}/_deps/dxc-src/bin/x64/dxil.dll
	${PLUME_DIR}/_deps/uwpdep-src/x64/bin/libuwp.dll
	${PLUME_DIR}/_deps/uwpdep-src/x64/bin/SDL2.dll

	Package.appxmanifest
)

set(BinLibs
	${PLUME_DIR}/${CMAKE_BUILD_TYPE}/plume.lib
	${PLUME_DIR}/examples/${CMAKE_BUILD_TYPE}/plume_example.lib

	${PLUME_DIR}/_deps/uwpdep-src/x64/lib/libuwp.lib
	${PLUME_DIR}/_deps/uwpdep-src/x64/lib/SDL2.lib

	d3d12
	dxgi
	dxguid
	gdi32
	OneCore
	WindowsApp
)
source_group("BinDeps" FILES ${BinDeps})

set_source_files_properties(${BinDeps} PROPERTIES
	VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "."
)

# Override deployment location for agility core
set_source_files_properties(${AgilityBins} PROPERTIES 
	VS_DEPLOYMENT_LOCATION "D3D12"
)

source_group("PkgAssets" FILES ${PkgAssets})

set_source_files_properties(${PkgAssets} PROPERTIES
	#VS_COPY_TO_OUT_DIR Always
	VS_DEPLOYMENT_CONTENT TRUE
	VS_DEPLOYMENT_LOCATION "Assets"
)

# Setup data deployment
file(GLOB_RECURSE DATA_FILES
	RELATIVE ${PLUME_DIR}
	${PLUME_DIR}/shaders/*
)

foreach(ASSET ${DATA_FILES})
	set(FULL_PATH "${PLUME_DIR}/${ASSET}")

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
 	${PLUME_DIR}/_deps/uwpdep-src/x64/include
)

target_link_libraries(${PROJECT_NAME} PRIVATE ${BinLibs})
set_target_properties(${PROJECT_NAME} PROPERTIES VS_WINDOWS_TARGET_PLATFORM_MIN_VERSION 10.0.19041.0)
