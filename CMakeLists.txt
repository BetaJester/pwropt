cmake_minimum_required(VERSION 3.18.0)

project(pwropt)

option(PWROPT_SIMPLE_TEST "Simple library compile to test" OFF)

# I wish generator expressions worked in "set", but they just don't.
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	set(pwr_wall /W4 /permissive-)
	set(pwr_werror /WX)
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	set(pwr_wall -Wall -Wextra -pedantic-errors)
	set(pwr_werror -Werror)
endif()

# Usage: pwropt_all_compile_options(<outputvar>)
# Get a list of all viable compile options
function(pwropt_all_compile_options outputvar)
	set(${outputvar} "pwr_wall;pwr_werror" PARENT_SCOPE)
endfunction()

# Usage: pwropt_parse_compile_options(<outputvar>)
# Get a list of pwropt parsed compile options
function(pwropt_parse_compile_options outputvar)
	pwropt_all_compile_options(pwropt_all)
	foreach(thing ${ARGN})
		if ("${thing}" IN_LIST pwropt_all)
			cmake_language(EVAL CODE "set(output \"\${${thing}}\")")
			set(out "${out};${output}")
		else()
			message("pwropt_parse: ${thing} not a pwropt expression, passing on as is")
			set(out "${out};${thing}")
		endif()
	endforeach()
	set(${outputvar} ${out} PARENT_SCOPE)
endfunction()

# Usage: pwropt_target_compile_options(<target> [BEFORE] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items1...] ...])
# Exactly like target_compile_options but parses pwropt options too.
function(pwropt_target_compile_options target)
	cmake_parse_arguments(
		PVAL
		"BEFORE"
		""
		"PUBLIC;PRIVATE;INTERFACE;"
		${ARGN}
	)
	pwropt_all_compile_options(pwropt_all)
	set(before "")
	if(${PVAL_BEFORE})
		set(before "BEFORE")
	endif()
	macro(eval_this visibility items)
		message("Processing ${items}")
		foreach(thing ${items})
			if ("${thing}" IN_LIST pwropt_all)
				cmake_language(EVAL CODE "set(output \"\${${thing}}\")")
				cmake_language(EVAL CODE "target_compile_options(${target} ${before} ${visibility} ${output})")
				message("target_compile_options(${target} ${before} ${visibility} ${output})")
			else()
				message("pwropt_target_compile_options : ${thing} not a pwropt expression, passing on as is")
				cmake_language(EVAL CODE "target_compile_options(${target} ${before} ${visibility} ${thing})")
				message("target_compile_options(${target} ${before} ${visibility} ${thing})")
			endif()
		endforeach()
	endmacro()
	if (DEFINED PVAL_PUBLIC)
		message("public args: ${PVAL_PUBLIC}")
		eval_this(PUBLIC "${PVAL_PUBLIC}")
	endif()
	if (DEFINED PVAL_PRIVATE)
		eval_this(PRIVATE ${PVAL_PRIVATE})
	endif()
	if (DEFINED PVAL_INTERFACE)
		eval_this(INTERFACE ${PVAL_INTERFACE})
	endif()
endfunction()

# Test that unknowns are passed on as is.
if (${PWROPT_SIMPLE_TEST})
	pwropt_parse_compile_options(outvar pwr_wall whatwhat test pwr_werror)
	message("pwropt_parse: ${outvar}")
	add_library(what "what.cpp")
	pwropt_target_compile_options(what PUBLIC pwr_wall pwr_werror /BAD)
endif()