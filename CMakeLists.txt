cmake_minimum_required(VERSION 3.3)

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

# Usage: pwropt_parse_option(<outputvar> <compile|link> <pwropt_option>)
# Decodes a pwropt option for current compiler, passes through unknown
# and errors on blocked.
# For internal use.
function(pwropt_parse_option outvar type)

	cmake_parse_arguments(
		PVAL
		""
		"OPTION"
		"COMPILERS"
		${ARGN}
	)

	if(DEFINED PVAL_COMPILERS AND NOT "${CMAKE_CXX_COMPILER_ID}" IN_LIST PVAL_COMPILERS)
		pwropt_message("Ignoring due to wrong compiler (${CMAKE_CXX_COMPILER_ID}) not in ${PVAL_COMPILERS}")
		return()
	endif()

	set(goodtypes compile link)
	string(TOLOWER ${type} type)
	if(NOT ${PVAL_TYPE} IN_LIST goodtypes)
		message(FATAL_ERROR "pwropt does not support type '${type}'")
		return()
	endif()

	# Generated from https://docs.google.com/spreadsheets/d/1Z82hdGryLYjyyD-t59o1PQKvZmJSPCnlzpx6ffjxWoo/edit?usp=sharing
	set(compilers "AppleClang;ARMClang;Clang;GNU;IntelLLVM;MSVC;XLClang")
	set(compiler_ids "1;1;1;2;1;0;1")
	set(pwroptions "pwr_wall;pwr_werror;pwr_pedantic;pwr_O0;pwr_O1;pwr_O2;pwr_O3;pwr_Odebug;pwr_Ofast;pwr_Osize;pwr_lto;pwr_no-exceptions;pwr_no-rtti;pwr_no-threadsafe-statics;pwr_cpp_20;pwr_cpp_17;pwr_cpp_14;pwr_cpp_11;pwr_lefticus")
	set(compile_opts "/W4;/WX;/permissive-;/Od;/O1;/O2;/O2;/Od;/Ot;/Os;/GL;;/GR-;/Zc:threadSafeInit-;-std:c++latest;-std:c++17;-std:c++14;-std:c++11; /W4 /w14242 /w14254 /w14263 /w14265  /w14287 /we4289  /w14296 /w14311 /w14545 /w14546 /w14547 /w14549 /w14555 /w14619 /w14640 /w14826 /w14905 /w14906 /w14928 /permissive-;-Wall -Wextra;-Werror;-pedantic-errors;-O0;-O1;-O2;-O3;-Og;-Ofast;-Os;-flto;-fno-exceptions;-fno-rtti;-fno-threadsafe-statics;-std=c++2a;-std=c++17;-std=c++14;-std=c++11;-Wall -Wextra -Wshadow -Wnon-virtual-dtor -Wold-style-cast -Wcast-align -Wunused -Woverloaded-virtual -Wpedantic -Wconversion -Wsign-conversion -Wnull-dereference -Wdouble-promotion -Wformat=2;-Wall -Wextra;-Werror;-pedantic-errors;-O0;-O1;-O2;-O3;-Og;-Ofast;-Os;-flto;-fno-exceptions;-fno-rtti;-fno-threadsafe-statics;-std=c++2a;-std=c++17;-std=c++14;-std=c++11;-Wall -Wextra -Wshadow -Wnon-virtual-dtor -Wold-style-cast -Wcast-align -Wunused -Woverloaded-virtual -Wpedantic -Wconversion -Wsign-conversion -Wnull-dereference -Wdouble-promotion -Wformat=2 -Wmisleading-indentation -Wduplicated-cond -Wduplicated-branches -Wlogical-op -Wuseless-cast")
	set(link_opts "/W4;/WX;;xxx;xxx;xxx;xxx;xxx;xxx;xxx;/LTCG;xxx;xxx;xxx;xxx;xxx;xxx;xxx;xxx;-Wall -Wextra;-Werror;-pedantic-errors;xxx;xxx;xxx;xxx;xxx;xxx;xxx;-flto;xxx;xxx;xxx;xxx;xxx;xxx;xxx;xxx;-Wall -Wextra;-Werror;-pedantic-errors;xxx;xxx;xxx;xxx;xxx;xxx;xxx;-flto;xxx;xxx;xxx;xxx;xxx;xxx;xxx;xxx")

	list(FIND compilers ${CMAKE_CXX_COMPILER_ID} compiler_index)
	if (${compiler_index} EQUAL -1)
		pwropt_message("pwropt_get_option: Count not find compiler")
		return()
	endif()

	list(GET compiler_ids ${compiler_index} compiler_id)
	pwropt_message("pwropt_get_option: compiler id is ${compiler_id}")

	list(FIND pwroptions ${PVAL_OPTION} id)
	if (${id} EQUAL -1)
		pwropt_message("pwropt_get_option: pwropt ${PVAL_OPTION} not found, passing through untouched")
		set(${outvar} ${PVAL_OPTION} PARENT_SCOPE)
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
		message(FATAL_ERROR "Not a supported type, somehow")
		return()
	endif()

	if ("${outthing}" STREQUAL "xxx")
		message(FATAL_ERROR "Blocked option - ${PVAL_OPTION} is not allowed as a ${type} option")
	endif()

	if(DEFINED outthing)
		string(REGEX REPLACE "[ ]" ";" outthing "${outthing}")
	endif()

	set(${outvar} ${outthing} PARENT_SCOPE)

endfunction()

# Usage: pwropt_parse_options(<outputvar> <compile|link>)
# Get a list of pwropt parsed options for a compile/link stage.
# For internal use.
function(pwropt_parse_options outputvar type)
	cmake_parse_arguments(
		PVAL
		""
		""
		"COMPILERS;OPTIONS"
		${ARGN}
	)
	set(out)
	foreach(thing ${PVAL_OPTIONS})
		pwropt_message("pwropt_parse_options parsing: ${thing}")
		pwropt_parse_option(result ${type} COMPILERS ${PVAL_COMPILERS} OPTION ${thing})
		if(DEFINED result)
			pwropt_message("pwropt_parse_x_options result: ${result}")
			list(APPEND out ${result})
			pwropt_message("pwropt_parse_x_options current: ${out}")
		endif()
	endforeach()
	set(${outputvar} ${out} PARENT_SCOPE)
endfunction()

# Usage: pwropt_target_options(<target> <type>)
# Actually processes and uses pwropt options.
# For internal use.
function(pwropt_target_options target type)

	pwropt_message("pwropt_target_options: ${target} ${type} ${ARGN}")

	set(goodtypes compile link)
	string(TOLOWER ${type} type)
	if(NOT ${type} IN_LIST goodtypes)
		message(FATAL_ERROR "pwropt does not support type '${type}'")
		return()
	endif()

	cmake_parse_arguments(
		PVAL
		"BEFORE"
		""
		"PUBLIC;PRIVATE;INTERFACE;COMPILERS"
		${ARGN}
	)

	set(before "")
	if(${PVAL_BEFORE})
		set(before "BEFORE")
	endif()

	macro(eval_this visibility items)
		pwropt_message("Processing ${items}")
		pwropt_parse_options(parseditems ${type} COMPILERS ${PVAL_COMPILERS} OPTIONS ${items})
		if(DEFINED parseditems)
			pwropt_message("Parsed items: ${parseditems}")
			if(${type} STREQUAL "compile")
				target_compile_options(${target} ${before} ${visibility} ${parseditems})
			elseif(${type} STREQUAL "link")
				target_link_options(${target} ${before} ${visibility} ${parseditems})
			endif()
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

# Usage: pwropt_target_compile_options(<target> [BEFORE] [COMPILERS <compiler1,...>] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items1...] ...])
# Exactly like target_compile_options but parses pwropt options too.
function(pwropt_target_compile_options target)
	pwropt_target_options(${target} COMPILE ${ARGN})
endfunction()

# Usage: pwropt_target_link_options(<target> [BEFORE] [COMPILERS <compiler1,...>] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items1...] ...])
# Exactly like target_link_options but parses pwropt options too.
function(pwropt_target_link_options target)
	pwropt_target_options(${target} LINK ${ARGN})
endfunction()

# Usage: pwropt_target_link_libraries(<target> [COMPILERS <compiler1,...>] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items1...] ...])
# Exactly like target_link_libraries (for now), with optional compiler restrictions
function(pwropt_target_link_libraries target)

	cmake_parse_arguments(
		PVAL
		""
		""
		"PUBLIC;PRIVATE;INTERFACE;COMPILERS"
		${ARGN}
	)

	if(DEFINED PVAL_COMPILERS AND NOT "${CMAKE_CXX_COMPILER_ID}" IN_LIST PVAL_COMPILERS)
		pwropt_message("Ignoring due to wrong compiler (${CMAKE_CXX_COMPILER_ID}) not in ${PVAL_COMPILERS}")
		return()
	endif()

	macro(eval_this visibility items)
		pwropt_message("Processing link libraries '${target}' '${visibility}' '${items}'")
		target_link_libraries(${target} ${visibility} ${items})
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

function(pwropt_target_cxx_latest target visibility)
	if("cxx_std_20" IN_LIST CMAKE_CXX_COMPILE_FEATURES)
		message("Building ${target} for C++20")
		target_compile_features(${target} ${visibility} cxx_std_20)
	elseif("cxx_std_17" IN_LIST CMAKE_CXX_COMPILE_FEATURES)
		message("Building ${target} for C++17")
		target_compile_features(${target} ${visibility} cxx_std_17)
	elseif("cxx_std_14" IN_LIST CMAKE_CXX_COMPILE_FEATURES)
		message("Building ${target} for C++14")
		target_compile_features(${target} ${visibility} cxx_std_14)
	elseif("cxx_std_11" IN_LIST CMAKE_CXX_COMPILE_FEATURES)
		message("Building ${target} for C++11")
		target_compile_features(${target} ${visibility} cxx_std_11)
	else()
		message(WARNING "pwropt cannot set features for latest C++, using compiler default")
	endif()
endfunction()

# Test that unknowns are passed on as is.
if (${PWROPT_SIMPLE_TEST})
	#pwropt_parse_compile_options(outvar pwr_wall whatwhat test pwr_werror)
	#pwropt_message("pwropt_parse: ${outvar}")
	add_executable(what "what.cpp")
	pwropt_target_cxx_latest(what PUBLIC)
	pwropt_target_compile_options(what PUBLIC pwr_wall pwr_werror pwr_lto)
	pwropt_target_compile_options(what COMPILERS GNU Clang PUBLIC pwr_pedantic -O2)
	pwropt_target_compile_options(what COMPILERS MSVC PUBLIC pwr_Odebug /permissive-)
	pwropt_target_link_options(what PUBLIC pwr_pedantic pwr_lto)
	pwropt_target_link_libraries(what COMPILERS GNU PUBLIC -pthread)
endif()