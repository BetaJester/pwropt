cmake_minimum_required(VERSION 3.18.0)

project(pwropt)

# $<$<CXX_COMPILER_ID:MSVC>:HERE> $<$<CXX_COMPILER_ID:Clang,GNU>:HERE>
set(pwr_wall $<$<CXX_COMPILER_ID:MSVC>:/W4 /permissive-> $<$<CXX_COMPILER_ID:Clang,GNU>:-Wall -Wextra -pedantic-errors>)
set(pwr_werror $<$<CXX_COMPILER_ID:MSVC>:/WX> $<$<CXX_COMPILER_ID:Clang,GNU>:-Werror>)

function(pwropt_target_compile_options target visibility)
	foreach(thing ${ARGN})
		cmake_language(EVAL CODE "set(output \"\${${thing}}\")")
		cmake_language(EVAL CODE "target_compile_options(${target} ${visibility} ${output})")
	endforeach()
endfunction()