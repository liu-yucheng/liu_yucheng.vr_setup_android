# // Copyright (C) 2024 Yucheng Liu. Under the GNU AGPL 3.0 License.
# // GNU AGPL 3.0 License available at: https://www.gnu.org/licenses/agpl-3.0.txt

# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# build native_app_glue as a static lib
set(APP_GLUE_DIR ${ANDROID_NDK}/sources/android/native_app_glue)
include_directories(${APP_GLUE_DIR})
add_library( app-glue STATIC ${APP_GLUE_DIR}/android_native_app_glue.c)

add_definitions(-DXR_USE_PLATFORM_ANDROID)
add_definitions(-DXR_USE_GRAPHICS_API_OPENGL_ES)
add_definitions(-DXR_USE_GRAPHICS_API_VULKAN)
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")

set(APP_CPP_NAME vr_setup_android_cpp)
set(LOADER_NAME openxr_loader)
set(APP_CPP_PATH app/${APP_CPP_NAME})
set(LOADER_PATH libs/${LOADER_NAME})

file(GLOB LOCAL_HEADERS "${APP_CPP_PATH}/*.h")
file(GLOB LOCAL_SOURCE "${APP_CPP_PATH}/*.cpp")
file(GLOB VULKAN_SHADERS "${APP_CPP_PATH}/vulkan_shaders/*.glsl")

LINK_DIRECTORIES(${LOADER_PATH}/${ANDROID_ABI})
include_directories(${LOADER_PATH}/include)

add_library(
	# Sets the name of the library.
	${APP_CPP_NAME}
	SHARED
	${LOCAL_SOURCE}
	${LOCAL_HEADERS}
	${VULKAN_SHADERS}
	${LOADER_PATH}/include/common/gfxwrapper_opengl.c
)

source_group("Headers" FILES ${LOCAL_HEADERS})
source_group("Shaders" FILES ${VULKAN_SHADERS})

if(VulkanHeaders_INCLUDE_DIRS)
    target_include_directories(
		${APP_CPP_PATH}
		PRIVATE
		${VulkanHeaders_INCLUDE_DIRS}
	)
endif()

if(Vulkan_LIBRARY)
    target_link_libraries(
		${APP_CPP_NAME}
		${Vulkan_LIBRARY}
	)
endif()

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries(
	# Specifies the target library.
	${APP_CPP_NAME}
	${LOADER_NAME}
	android
	vulkan
	EGL
	GLESv3
	app-glue
	log
)
