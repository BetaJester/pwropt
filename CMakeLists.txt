cmake_minimum_required(VERSION 3.18.0)

project(pwropt)

option(PWROPT_SIMPLE_TEST "Simple code/compile to test" OFF)
option(PWROPT_NOISY "Output too much information" OFF)

macro(pwropt_message msg)
	if(${PWROPT_NOISY})
		message(${msg})
	endif()
endmacro()

function(pwropt_get_option outvar type option)

	# Like a database, but not and also shit.
	set(compilers "AppleClang;ARMClang;Clang;GNU;IntelLLVM;MSVC;XLClang")
	set(compiler_ids "1;1;1;1;1;0;1")
	set(pwroptions "pwr_wall;pwr_werror;pwr_pedantic")
	set(compile_opts "/W4;/WX;/permissive-;-Wall -Wextra;-Werror;-pedantic-errors")
	set(link_opts "/W4;/WX;;-Wall -Wextra;-Werror;-pedantic-errors")

	list(FIND compilers ${CMAKE_CXX_COMPILER_ID} compiler_index)
	if (${compiler_index} EQUAL -1)
		pwropt_message("pwropt_get_option: Count not find compiler")
		return()
	endif()

	list(GET compiler_ids ${compiler_index} compiler_id)
	pwropt_message("pwropt_get_option: compiler id is ${compiler_id}")

	list(FIND pwroptions ${option} id)
	if (${id} EQUAL -1)
		pwropt_message("pwropt_get_option: pwropt ${option}")
		return()
	endif()

	list(LENGTH pwroptions len)
	math(EXPR index "${compiler_id} * ${len} + ${id}")
	pwropt_message("pwropt_get_option: option index is ${compiler_id} * ${len} + ${id} (${index})")
	
	if("${type}" STREQUAL "compile")
		list(GET compile_opts ${index} outthing)
	elseif("${type}" STREQUAL "link")
		list(GET link_opts ${index} outthing)
	else()
		return()
	endif()

	pwropt_message("pwropt_get_option: output should be ${outthing}")
	set(${outvar} ${outthing} PARENT_SCOPE)

endfunction()

# I wish generator expressions worked in "set", but they just don't.
if (${CMAKE_CXX_COMPILER_ID} IN_LIST "MSVC")
	set(pwr_wall /W4 /permissive-)
	set(pwr_werror /WX)
elseif(${CMAKE_CXX_COMPILER_ID} IN_LIST "Clang;GNU")
	set(pwr_wall -Wall -Wextra -pedantic-errors)
	set(pwr_werror -Werror)
endif()

# Usage: pwropt_all_compile_options(<outputvar>)
# Get a list of all viable compile options
function(pwropt_all_compile_options outputvar)
	set(${outputvar} "pwr_wall;pwr_werror" PARENT_SCOPE)
endfunction()

# Usage: pwropt_all_link_options(<outputvar>)
# Get a list of all viable link options
function(pwropt_all_link_options outputvar)
	set(${outputvar} "pwr_lto" PARENT_SCOPE)
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
			pwropt_message("pwropt_parse: ${thing} not a pwropt expression, passing on as is")
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
		pwropt_message("Processing ${items}")
		pwropt_parse_compile_options(parseditems ${items})
		pwropt_message("Parsed items: ${parseditems}")
		cmake_language(EVAL CODE "target_compile_options(${target} ${before} ${visibility} ${parseditems})")
		pwropt_message("target_compile_options(${target} ${before} ${visibility} ${parseditems})")
	endmacro()
	if (DEFINED PVAL_PUBLIC)
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
	pwropt_get_option(outvar "compile" "pwr_werror")
	if(DEFINED outvar)
		pwropt_message("pwropt_get_option result: ${outvar}")
	else()
		pwropt_message("pwropt_get_option undefined")
	endif()
	pwropt_get_option(outvar "link" "pwr_pedantic")
	if(DEFINED outvar)
		pwropt_message("pwropt_get_option result (link): ${outvar}")
	else()
		pwropt_message("pwropt_get_option undefined (link)")
	endif()
	pwropt_parse_compile_options(outvar pwr_wall whatwhat test pwr_werror)
	pwropt_message("pwropt_parse: ${outvar}")
	add_library(what "what.cpp")
	pwropt_target_compile_options(what PUBLIC pwr_wall pwr_werror /BAD PRIVATE pvt PUBLIC ahoy)
endif()