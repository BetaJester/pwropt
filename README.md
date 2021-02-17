# pwropt
A one file improvement to target_compile_options

pwropt allows cross platform compile options in CMake, so you don't have to fiddle with compiler specific flags (most of the time).

* [Requirements](#requirements)
* [Usage](#usage)
* [List of Flags](#list-of-flags)

## Requirements

CMake 3.18 must be used for access to `cmake_language`, for the moment.

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
pwropt_target_compile_options(someapp PRIVATE pwr_wall pwr_werror)
```

## List of Flags

### pwr_wall

Enables all warnings possible.

### pwr_werror

Turns most warnings into errors.