message("Beginning shader compile")

if (${CMAKE_HOST_SYSTEM_PROCESSOR} STREQUAL "AMD64")
	set(GLSL_VALIDATOR "$ENV{VULKAN_SDK}/Bin/glslangValidator.exe")
else()
	set(GLSL_VALIDATOR "$ENV{VULKAN_SDK}/Bin32/glslangValidator.exe")
endif()

message("Finding shader source files in: ${CMAKE_CURRENT_SOURCE_DIR}")

file(GLOB_RECURSE GLSL_SOURCE_FILES
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.frag"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.vert"
	)

message(Found: ${GLSL_SOURCE_FILES})

if(NOT DEFINED SHADER_OUTPUT_DIR)
    message(FATAL_ERROR "The SHADER_OUTPUT_DIR value is not defined! Please set it.")
endif()

message("Output directory: ${CMAKE_SOURCE_DIR}")

foreach(GLSL ${GLSL_SOURCE_FILES})
	get_filename_component(FILE_NAME ${GLSL} NAME)
	set(SPIRV "${SHADER_OUTPUT_DIR}/shaders/${FILE_NAME}.spv")
	add_custom_command(
		OUTPUT ${SPIRV}
		COMMAND ${CMAKE_COMMAND} -E make_directory "${SHADER_OUTPUT_DIR}/shaders"
		COMMAND ${GLSL_VALIDATOR} -V ${GLSL} -o ${SPIRV}
		DEPENDS ${GLSL})
	list(APPEND SPIRV_BINARY_FILES ${SPIRV})
endforeach(GLSL)

add_custom_target(
	Shaders 
	DEPENDS ${SPIRV_BINARY_FILES}
	)
