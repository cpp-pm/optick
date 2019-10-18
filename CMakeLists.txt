cmake_minimum_required(VERSION 3.2)
project(Optick LANGUAGES CXX)


# Sources
file(GLOB OPTICK_SRC "src/*.*")
source_group("OptickCore" FILES ${OPTICK_SRC})


# Enabled
option(OPTICK_ENABLED "Enable profiling with Optick" ON)
if(NOT OPTICK_ENABLED)
	message(STATUS "Optick is disabled")
	# add dummy target as a replacement
	add_library(OptickCore STATIC ${OPTICK_SRC})
	target_include_directories(OptickCore PUBLIC "src")
	target_compile_definitions(OptickCore PUBLIC USE_OPTICK=0)
	return()
endif()


# Options
option(OPTICK_USE_VULKAN "Built-in support for Vulkan" OFF)
option(OPTICK_USE_D3D12 "Built-in support for DirectX 12" OFF)


# OptickCore
add_library(OptickCore STATIC ${OPTICK_SRC})
target_include_directories(OptickCore PUBLIC "src")
target_compile_definitions(OptickCore PUBLIC OPTICK_EXPORTS=1)
if(OPTICK_USE_VULKAN)
	message(STATUS "Optick uses Vulkan")
	target_include_directories(OptickCore PRIVATE "$(VULKAN_SDK)/Include")
else()
	target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU_VULKAN=0)
endif()
if(OPTICK_USE_D3D12)
	message(STATUS "Optick uses DirectX 12")
else()
	target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU_D3D12=0)
endif()
if(OPTICK_USE_D3D12 OR OPTICK_USE_VULKAN)
	target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU=1)
else()
	target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU=0)
endif()

if(MSVC)
	# temporary solution to unblock C++17 users
	target_compile_definitions(OptickCore PRIVATE _SILENCE_ALL_CXX17_DEPRECATION_WARNINGS)
endif()
