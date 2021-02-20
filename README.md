# pwropt
A one file improvement to target_compile_options

pwropt allows cross platform compile/link options in CMake, so you don't have to fiddle with compiler specific flags (most of the time).

* [Requirements](#requirements)
* [Usage](#usage)
* [List of Flags](#list-of-flags)
* [License](#license)

## Requirements

CMake 3.3 or above must be used. If you aren't using much higher you should.

## Usage

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