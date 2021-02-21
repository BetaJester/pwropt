# pwropt
A one file improvement to target_compile_options

pwropt allows cross platform compile/link options in CMake, so you don't have to fiddle with compiler specific flags (most of the time). Specifically made with C/C++ in mind.

* [Requirements](#requirements)
* [Usage](#usage)
* [List of Flags](#list-of-flags)
* [Supported Compilers](#supported-compilers)
* [License](#license)

## Requirements

CMake 3.3 or above must be used. If you aren't using much higher you should.

## Usage

Same functionality as `target_compile_options` and `target_link_options` have in CMake, with some differences.

All `pwr_<name>` flags are translated to the compiler equivalent - this is the main power of the library, you no longer have to remember differing flags per compiler. Flags that are not recognized  by pwropt pass through unchanged so you can migrate to pwropt easily, and use flags pwropt doesn't support.

Also, you can specify the compilers you want these flags to apply to (Using flags from [the CMake documentaton](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_ID.html). This is largely because generator expressions wont work in user defined functions, but it is also neater.

The parsing functions used internally by these functions are

```cmake
pwropt_target_link_options(<target> [BEFORE]
  [<COMPILERS> [compiler1...]]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...]) 
```

```cmake
pwropt_target_compile_options(<target> [BEFORE]
  [<COMPILERS> [compiler1...]]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

```cmake
pwropt_parse_options(<outputvar> <COMPILE|LINK>
  [<COMPILERS> [compiler1...]]
  <OPTIONS> [items1...])
```

```cmake
pwropt_parse_option(<outputvar> <COMPILE|LINK>
  [<COMPILERS> [compiler1...]]
  <OPTION> <item>
)
```

---

Very basic usage to give the idea.

```cmake
FetchContent_Declare(
	pwropt
	GIT_REPOSITORY https://github.com/BetaJester/pwropt.git
	GIT_TAG main
)
FetchContent_MakeAvailable(pwropt)
add_subdirectory(${pwropt_SOURCE_DIR})

add_executable(someapp "main.cpp")
pwropt_target_compile_options(someapp PRIVATE pwr_wall pwr_werror pwr_lto)		# All compilers
pwropt_target_compile_options(someapp COMPILERS Clang GNU PRIVATE pwr_Odebug)	# Just Clang and GCC.
pwropt_target_link_options(someapp PRIVATE pwr_lto)
```

## List of Flags

Currently the best source of flags is from the [spreadsheet](https://docs.google.com/spreadsheets/d/1Z82hdGryLYjyyD-t59o1PQKvZmJSPCnlzpx6ffjxWoo/edit?usp=sharing) used to generate the data.

## Supported Compilers

The [spreadsheet](https://docs.google.com/spreadsheets/d/1Z82hdGryLYjyyD-t59o1PQKvZmJSPCnlzpx6ffjxWoo/edit?usp=sharing) used to generate the data also lists compilers, but at the moment MSVC, Clang (and variants like AppleClang) and GCC are supported explicitly. There is still a *lot* of work to do building the database to run this at the quality I hope for.

## License

MIT License

Copyright (c) 2021 Glenn Duncan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.