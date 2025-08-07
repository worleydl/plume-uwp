set(CMAKE_CXX_STANDARD 17)

# Enable SDL Vulkan integration
set(PLUME_SDL_VULKAN_ENABLED ON CACHE BOOL "Enable SDL Vulkan integration" FORCE)

if(NOT UWP)
    # Find SDL2 (required for examples)
    find_package(SDL2 REQUIRED)
else()
    include(FetchContent)
    FetchContent_Declare(
        uwpdep
        GIT_REPOSITORY https://github.com/worleydl/uwp-dep.git
        GIT_TAG 8884b71dba77c129bb3a4f05db3b2f68fe763e55
    )
    FetchContent_MakeAvailable(uwpdep)
    set(SDL2_LIBRARIES SDL2)
endif()

# Initialize shader compilation
include(${CMAKE_SOURCE_DIR}/examples/cmake/PlumeShaders.cmake)
plume_shaders_init()

# Add example subdirectories
add_subdirectory(triangle)
add_subdirectory(cube)
