###
##
# @NOTE:
# This is an edited CMake, different from the original CMake file found in RmlUI's repo.
# Mainly, the Lua bindings and Tracy logic is taken out.
# Samples, backends, and tests are all removed.
# All dependencies are note retrieved using `FetchContent` instead of vcpkg.
##
###

cmake_minimum_required(VERSION 3.10...3.27)
project("RmlUi"
	VERSION "6.2"
	DESCRIPTION "C++ user interface library based on the HTML and CSS standards"
	LANGUAGES "C" "CXX"
)

### FetchContent ###
############################################################
include(FetchContent)
############################################################

### Options ###
############################################################
option(BUILD_SHARED_LIBS "CMake standard option. Choose whether to build shared RmlUi libraries." OFF)
option(RMLUI_BACKEND_SIMULATE_TOUCH "Simulate touch events with mouse events" OFF)
option(RMLUI_LUA_BINDINGS "Build Lua bindings." OFF)
option(RMLUI_LOTTIE_PLUGIN "Enable plugin for Lottie animations. Requires the rlottie library." OFF)
option(RMLUI_SVG_PLUGIN "Enable plugin for SVG images. Requires the lunasvg library." OFF)
option(RMLUI_MATRIX_ROW_MAJOR "Use row-major matrices. Column-major matrices are used by default." OFF)
option(RMLUI_THIRDPARTY_CONTAINERS "Enable integrated third-party containers for improved performance, rather than their standard library counterparts." ON)
option(RMLUI_CUSTOM_RTTI "Build RmlUi with a custom implementation of run-time type information (RTTI). When enabled, all usage of language RTTI features such as dynamic_cast will be disabled." OFF)
option(RMLUI_PRECOMPILED_HEADERS "Enable precompiled headers for RmlUi." ON)
option(RMLUI_COMPILER_OPTIONS "Enable recommended compiler-specific options for the project, such as for supported warning level, standards conformance, and multiprocess builds. Turn off for full control over compiler flags." ON)
option(RMLUI_CUSTOM_CONFIGURATION "Customize the RmlUi configuration file to override the default configuration and types." OFF)
############################################################

### Project variables ###
############################################################

# Configuring the font engine

set(RMLUI_VERSION_RELEASE FALSE)
set(RMLUI_FONT_ENGINE "freetype" CACHE STRING "Font engine to build RmlUi with. If set to \"none\", RmlUi won't build with any of the included font engines and some samples might be disabled.")
set_property(CACHE RMLUI_FONT_ENGINE PROPERTY STRINGS ${RMLUI_FONT_ENGINE_OPTIONS})

if(RMLUI_FONT_ENGINE STREQUAL "none")
	set(RMLUI_FONT_ENGINE_ENABLED FALSE)
else()
	set(RMLUI_FONT_ENGINE_ENABLED TRUE)
endif()

# Config files

set(RMLUI_CUSTOM_CONFIGURATION_FILE "" CACHE STRING "Custom configuration file to be included in place of <RmlUi/Config/Config.h>.")
set(RMLUI_CUSTOM_INCLUDE_DIRS "" CACHE STRING "Extra include directories (use with RMLUI_CUSTOM_CONFIGURATION_FILE).")
set(RMLUI_CUSTOM_LINK_LIBRARIES "" CACHE STRING "Extra link libraries (use with RMLUI_CUSTOM_CONFIGURATION_FILE).")
if(RMLUI_CUSTOM_CONFIGURATION)
	mark_as_advanced(CLEAR RMLUI_CUSTOM_CONFIGURATION_FILE RMLUI_CUSTOM_INCLUDE_DIRS RMLUI_CUSTOM_LINK_LIBRARIES)
else()
	mark_as_advanced(FORCE RMLUI_CUSTOM_CONFIGURATION_FILE RMLUI_CUSTOM_INCLUDE_DIRS RMLUI_CUSTOM_LINK_LIBRARIES)
	if(RMLUI_CUSTOM_CONFIGURATION_FILE OR RMLUI_CUSTOM_INCLUDE_DIRS OR RMLUI_CUSTOM_LINK_LIBRARIES)
		message(STATUS "RMLUI_CUSTOM_CONFIGURATION is disabled, but custom configuration variables are set. They will have no effect.")
	endif()
endif()
############################################################

### Dependencies ###
############################################################

# Freetype

if(RMLUI_FONT_ENGINE STREQUAL "freetype")
	FetchContent_Declare(
		freetype
		GIT_REPOSITORY https://github.com/freetype/freetype.git
		GIT_TAG        fc9cc5038e05edceec3d0f605415540ac76163e9
	)
	FetchContent_MakeAvailable(freetype)
endif()

# Lottie

if(RMLUI_LOTTIE_PLUGIN)
	FetchContent_Declare(
		rlottie
		URL https://github.com/Samsung/rlottie/archive/refs/tags/v0.2.zip
	)
	FetchContent_MakeAvailable(rlottie)
endif()

# SVG

if(RMLUI_SVG_PLUGIN)
	FetchContent_Declare(
		lunasvg
		URL https://github.com/sammycage/lunasvg/archive/refs/tags/v3.5.0.zip
	)
	FetchContent_MakeAvailable(lunasvg)
endif()
############################################################

### Utils configs ###
############################################################
include("CMake/Utilities.cmake")
include("CMake/RuntimeUtilities.cmake")

setup_binary_output_directories()
setup_runtime_dependency_set_arg()
############################################################

### Sub-directories ###
############################################################
add_subdirectory("Source")
############################################################

### Compile ###
############################################################
include(CMakePackageConfigHelpers)
configure_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/CMake/RmlUiConfig.cmake.in"
	"${CMAKE_CURRENT_BINARY_DIR}/install/RmlUiConfig.cmake"
	@ONLY
)

add_library(rmlui INTERFACE)
add_library(RmlUi::RmlUi ALIAS rmlui)

target_link_libraries(rmlui INTERFACE rmlui_core rmlui_debugger)
set_target_properties(rmlui PROPERTIES EXPORT_NAME "RmlUi")

write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/install/RmlUiConfigVersion.cmake"
	COMPATIBILITY ExactVersion
)

if(RMLUI_IS_ROOT_PROJECT)
	# Export build tree targets if RmlUi is the top-level project.
	export(EXPORT RmlUiTargets
		NAMESPACE RmlUi::
		FILE RmlUiConfig.cmake
	)
endif()
############################################################
