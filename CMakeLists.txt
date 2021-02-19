cmake_minimum_required(VERSION 3.18.0)

project(pwropt)

option(PWROPT_SIMPLE_TEST "Simple code/compile to test" OFF)
option(PWROPT_NOISY "Output too much information" OFF)

# Usage: pwropt_message(<text>)
# Output message if PWROPT_NOISY is set.
# For internal use.
macro(pwropt_message msg)
	if(${PWROPT_NOISY})
		message(${msg})
	endif()
endmacro()

# Usage: pwropt_get_option(<outputvar> <compile|link> <pwropt_option>)
# Decodes a pwropt option for current compiler, passes through unknown
# and errors on blocked.
# For internal use.
function(pwropt_get_option outvar type option)

	# Generated from https://docs.google.com/spreadsheets/d/1Z82hdGryLYjyyD-t59o1PQKvZmJSPCnlzpx6ffjxWoo/edit?usp=sharing
	set(compilers "AppleClang;ARMClang;Clang;GNU;IntelLLVM;MSVC;XLClang")
	set(compiler_ids "1;1;1;1;1;0;1")
	set(pwroptions "pwr_wall;pwr_werror;pwr_pedantic;pwr_O0;pwr_O1;pwr_O2;pwr_O3;pwr_Odebug;pwr_Ofast;pwr_Osize;pwr_lto")
	set(compile_opts "/W4;/WX;/permissive-;/Od;/O1;/O2;/O2;/Od;/Ot;/Os;/GL;-Wall -Wextra;-Werror;-pedantic-errors;-O0;-O1;-O2;-O3;-Og;-Ofast;-Os;-flto")
	set(link_opts "/W4;/WX;;xxx;xxx;xxx;xxx;xxx;xxx;xxx;/LTCG;-Wall -Wextra;-Werror;-pedantic-errors;xxx;xxx;xxx;xxx;xxx;xxx;xxx;-flto")

	list(FIND compilers ${CMAKE_CXX_COMPILER_ID} compiler_index)
	if (${compiler_index} EQUAL -1)
		pwropt_message("pwropt_get_option: Count not find compiler")
		return()
	endif()

	list(GET compiler_ids ${compiler_index} compiler_id)
	pwropt_message("pwropt_get_option: compiler id is ${compiler_id}")

	list(FIND pwroptions ${option} id)
	if (${id} EQUAL -1)
		pwropt_message("pwropt_get_option: pwropt ${option} not found, passing through untouched")
		set(${outvar} ${option} PARENT_SCOPE)
		return()
	endif()

	list(LENGTH pwroptions len)
	math(EXPR index "${compiler_id} * ${len} + ${id}")
	pwropt_message("pwropt_get_option: option index is ${compiler_id} * ${len} + ${id} (${index})")
	
	pwropt_message("pwropt_get_option: looking up as type ${type}")
	if("${type}" STREQUAL "compile")
		list(GET compile_opts ${index} outthing)
	elseif("${type}" STREQUAL "link")
		list(GET link_opts ${index} outthing)
	else()
		return()
	endif()

	if ("${outthing}" STREQUAL "xxx")
		message(FATAL_ERROR "Blocked option - ${option} is not allowed as a ${type} option")
	endif()

	set(${outvar} ${outthing} PARENT_SCOPE)

endfunction()

# Usage: pwropt_parse_x_options(<outputvar> <compile|link>)
# Get a list of pwropt parsed options for a compile/link stage.
# For internal use.
function(pwropt_parse_x_options outputvar type)
	set(out)
	foreach(thing ${ARGN})
		pwropt_message("pwropt_parse_x_options parsing: ${thing}")
		pwropt_get_option(result "${type}" ${thing})
		if(DEFINED result)
			pwropt_message("pwropt_parse_x_options result: ${result}")
			list(APPEND out ${result})
			pwropt_message("pwropt_parse_x_options current: ${out}")
		endif()
	endforeach()
	set(${outputvar} ${out} PARENT_SCOPE)
endfunction()

# Usage: pwropt_target_x_options(<target> <type>)
# Actually processes and uses pwropt options.
# For internal use.
function(pwropt_target_x_options target type)
	cmake_parse_arguments(
		PVAL
		"BEFORE"
		""
		"PUBLIC;PRIVATE;INTERFACE;"
		${ARGN}
	)
	set(before "")
	if(${PVAL_BEFORE})
		set(before "BEFORE")
	endif()
	macro(eval_this visibility items)
		pwropt_message("Processing ${items}")
		pwropt_parse_x_options(parseditems "${type}" ${items})
		if(DEFINED parseditems)
			pwropt_message("Parsed items: ${parseditems}")
			cmake_language(EVAL CODE "target_${type}_options(${target} ${before} ${visibility} ${parseditems})")
			pwropt_message("target_${type}_options(${target} ${before} ${visibility} ${parseditems})")
		endif()
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

# Usage: pwropt_target_compile_options(<target> [BEFORE] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items1...] ...])
# Exactly like target_compile_options but parses pwropt options too.
function(pwropt_target_compile_options target)
	pwropt_target_x_options(${target} "compile" "${ARGN}")
endfunction()

# Usage: pwropt_target_compile_options(<target> [BEFORE] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items1...] ...])
# Exactly like target_compile_options but parses pwropt options too.
function(pwropt_target_link_options target)
	pwropt_target_x_options(${target} "link" "${ARGN}")
endfunction()

# Test that unknowns are passed on as is.
if (${PWROPT_SIMPLE_TEST})
	#pwropt_parse_compile_options(outvar pwr_wall whatwhat test pwr_werror)
	#pwropt_message("pwropt_parse: ${outvar}")
	add_executable(what "what.cpp")
	pwropt_target_compile_options(what PUBLIC pwr_wall pwr_werror pwr_lto)
	pwropt_target_link_options(what PUBLIC pwr_pedantic pwr_lto)
endif()