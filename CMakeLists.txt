cmake_minimum_required(VERSION 3.18.0)

project(pwropt)

# I wish generator expressions worked in "set", but they just don't.
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	set(pwr_wall /W4 /permissive-)
	set(pwr_werror /WX)
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	set(pwr_wall -Wall -Wextra -pedantic-errors)
	set(pwr_werror -Werror)
endif()

function(pwropt_target_compile_options target visibility)
	foreach(thing ${ARGN})
		cmake_language(EVAL CODE "set(output \"\${${thing}}\")")
		cmake_language(EVAL CODE "target_compile_options(${target} ${visibility} ${output})")
	endforeach()
endfunction()