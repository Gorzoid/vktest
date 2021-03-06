cmake_minimum_required(VERSION 3.10)

include("CMake/HunterGate.cmake")

option(HUNTER_NO_TOOLCHAIN_ID_RECALCULATION "Speeds up configure time." ON)

HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.48.tar.gz"
    SHA1 "515a12255189d9f85e88174c56ea5d1ed3031d40"
)

project(vktest)

set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMake")
include(cotire)

# Source Files

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS YES)

list(APPEND SOURCE_FILES main.cpp)

list(APPEND SOURCE_FILES globals.cpp)
list(APPEND HEADER_FILES globals.h)

list(APPEND SOURCE_FILES window.cpp)
list(APPEND HEADER_FILES window.h)

list(APPEND SOURCE_FILES renderer.cpp)
list(APPEND HEADER_FILES renderer.h)

list(APPEND SOURCE_FILES scene.cpp)
list(APPEND HEADER_FILES scene.h)

list(APPEND SOURCE_FILES camera.cpp)
list(APPEND HEADER_FILES camera.h)

list(APPEND SOURCE_FILES buffer.cpp)
list(APPEND HEADER_FILES buffer.h)

list(APPEND SOURCE_FILES shader.cpp)
list(APPEND HEADER_FILES shader.h)

list(APPEND SOURCE_FILES pipeline.cpp)
list(APPEND HEADER_FILES pipeline.h)

list(APPEND SOURCE_FILES mesh.cpp)
list(APPEND HEADER_FILES mesh.h)

list(APPEND HEADER_FILES stdafx.h)

add_executable(${PROJECT_NAME} stdafx.cpp ${HEADER_FILES})
target_sources(${PROJECT_NAME} PUBLIC ${SOURCE_FILES})

if (WIN32)
add_definitions("-DNOMINMAX")
endif()

add_definitions("-D_USE_MATH_DEFINES")


# Libraries

hunter_add_package(glfw)

find_package(glfw3 3.3 REQUIRED)
#if(GLFW3_FOUND)
	include_directories(${GLFW3_INCLUDE_PATH})
	target_link_libraries(${PROJECT_NAME} glfw)
#endif()


#hunter_add_package(Boost COMPONENTS hana)
#find_package(boost REQUIRED COMPONENTS hana)
#if(Boost_FOUND)
#	include_directories(${Boost_INCLUDE_DIRS})
#	target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES})
#	#add_definitions("-D_SILENCE_ALL_CXX17_DEPRECATION_WARNINGS")
#endif()

find_package(Vulkan REQUIRED)
#if (VULKAN_FOUND)
	message(STATUS "Found Vulkan, Including and Linking now")
	include_directories(${Vulkan_INCLUDE_DIRS})
	target_link_libraries(${PROJECT_NAME} ${Vulkan_LIBRARIES})
#endif (VULKAN_FOUND)

hunter_add_package(glm)
find_package(glm REQUIRED)
#if (GLM_FOUND)
	message(STATUS "Found GLM, Including ${GLM_INCLUDE_DIRS}")
	include_directories(${GLM_INCLUDE_DIRS})
#endif ()

include_directories(libraries/meta/include)
include_directories(libraries/include/)

#########################
#						#
#	   GLSL Shaders		#
#						#
#########################

set(GLSL_VALIDATOR "glslangValidator")
file(GLOB_RECURSE GLSL_SOURCE_FILES
	"shaders/*.frag"
	"shaders/*.vert"
	"shaders/*.comp"
	)

foreach(GLSL ${GLSL_SOURCE_FILES})
	get_filename_component(FILE_NAME ${GLSL} NAME)
	get_filename_component(FILE_EXT ${GLSL} EXT)
	message(STATUS "Found ${FILE_EXT} shader '${FILE_NAME}'")
	set(SPIRV "${PROJECT_BINARY_DIR}/shaders/${FILE_NAME}.spv")
	add_custom_command(
		OUTPUT ${SPIRV}
		COMMAND ${CMAKE_COMMAND} -E make_directory "${PROJECT_BINARY_DIR}/shaders/"
		COMMAND ${GLSL_VALIDATOR} -V ${GLSL} -o ${SPIRV}
		DEPENDS ${GLSL})
	list(APPEND SPIRV_BINARY_FILES ${SPIRV})
endforeach(GLSL)

add_custom_target(
	SpirvShaders
	DEPENDS ${SPIRV_BINARY_FILES}
	SOURCES ${GLSL_SOURCE_FILES}
	)
source_group("Shaders" FILES ${GLSL_SOURCE_FILES})

add_dependencies(${PROJECT_NAME} SpirvShaders)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:${PROJECT_NAME}>/shaders/"
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${PROJECT_BINARY_DIR}/shaders"
        "$<TARGET_FILE_DIR:${PROJECT_NAME}>/shaders"
        )

#cotire(vktest)
