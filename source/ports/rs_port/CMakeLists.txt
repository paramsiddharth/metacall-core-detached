# Check if this port is enabled
if(NOT OPTION_BUILD_PORTS OR NOT OPTION_BUILD_PORTS_RS)
	return()
endif()

#
# Port name and options
#

file(READ ${CMAKE_CURRENT_SOURCE_DIR}/rust-toolchain TARGET_TOOLCHAIN)
string(STRIP "${TARGET_TOOLCHAIN}" TARGET_TOOLCHAIN)

find_package(Rust COMPONENTS ${TARGET_TOOLCHAIN})

if(NOT Rust_FOUND)
	message(STATUS "Rust not found")
	return()
endif()

# Target name
set(target rs_port)

# Exit here if required dependencies are not met
message(STATUS "Port ${target}")

if(CMAKE_BUILD_TYPE STREQUAL "Release")
	set(TARGET_BUILD_TYPE "--release")
else()
	set(TARGET_BUILD_TYPE)
endif()

add_custom_target(${target} ALL
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
	COMMAND ${Rust_CARGO_EXECUTABLE} build ${TARGET_BUILD_TYPE}
)

#
# Dependecies
#

add_dependencies(${target}
	${META_PROJECT_NAME}::metacall
)

#
# Define test
#

if(OPTION_BUILD_SANITIZER OR OPTION_BUILD_THREAD_SANITIZER)
	# TODO: This test fails when run with sanitizers:
	#
	# Address Sanitizer:
	# ASan runtime does not come first in initial library list; you should either link runtime to your application or manually preload it with LD_PRELOAD.
	# For solving the issue compile the test with sanitizers when enabled or preload asan
	#
	# Thread Sanitizer:
	# process didn't exit successfully: `/usr/local/metacall/source/ports/rs_port/target/debug/deps/metacall-f8cdc0b6dd115278` (signal: 11, SIGSEGV: invalid memory reference)
	return()
endif()

add_test(NAME ${target}
	COMMAND ${CMAKE_COMMAND} -E env CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} CMAKE_BINARY_DIR=${CMAKE_BINARY_DIR} RUSTFLAGS=-Zmacro-backtrace ${Rust_CARGO_EXECUTABLE} test
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

set_property(TEST ${target}
	PROPERTY LABELS ${target}
)

include(TestEnvironmentVariables)
include(Portability)

project_library_path(TEST_LIB_PATH
	${PROJECT_OUTPUT_DIR}
)

test_environment_variables(${target}
	""
	${TESTS_ENVIRONMENT_VARIABLES}
	"${PROJECT_LIBRARY_PATH_NAME}=${TEST_LIB_PATH}"
	"RUST_BACKTRACE=1"
)
