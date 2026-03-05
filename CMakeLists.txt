cmake_minimum_required(VERSION 3.28)
project(q3vm LANGUAGES C)

set(CMAKE_C_STANDARD 23)
set(CMAKE_C_STANDARD_REQUIRED TRUE)

# main vm library
add_library(q3vmlib STATIC ${PROJECT_SOURCE_DIR}/src/vm/vm.c)
set_target_properties(q3vmlib PROPERTIES OUTPUT_NAME q3vm)
add_library(q3vm::q3vm ALIAS q3vmlib)
target_include_directories(q3vmlib PUBLIC ${PROJECT_SOURCE_DIR}/src/vm)

# preprocessor
add_executable(q3cpp
	${PROJECT_SOURCE_DIR}/lcc/cpp/cpp.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/cpplex.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/eval.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/getopt.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/hideset.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/include.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/macro.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/nlist.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/tokens.c
	${PROJECT_SOURCE_DIR}/lcc/cpp/unix.c
)

# code generator
add_executable(q3rcc
	${PROJECT_SOURCE_DIR}/lcc/src/alloc.c
	${PROJECT_SOURCE_DIR}/lcc/src/bind.c
	${PROJECT_SOURCE_DIR}/lcc/src/bytecode.c
	${PROJECT_SOURCE_DIR}/lcc/src/dag.c
	${PROJECT_SOURCE_DIR}/lcc/src/dagcheck.c
	${PROJECT_SOURCE_DIR}/lcc/src/decl.c
	${PROJECT_SOURCE_DIR}/lcc/src/enode.c
	${PROJECT_SOURCE_DIR}/lcc/src/error.c
	${PROJECT_SOURCE_DIR}/lcc/src/event.c
	${PROJECT_SOURCE_DIR}/lcc/src/expr.c
	${PROJECT_SOURCE_DIR}/lcc/src/gen.c
	${PROJECT_SOURCE_DIR}/lcc/src/init.c
	${PROJECT_SOURCE_DIR}/lcc/src/inits.c
	${PROJECT_SOURCE_DIR}/lcc/src/input.c
	${PROJECT_SOURCE_DIR}/lcc/src/lex.c
	${PROJECT_SOURCE_DIR}/lcc/src/list.c
	${PROJECT_SOURCE_DIR}/lcc/src/main.c
	${PROJECT_SOURCE_DIR}/lcc/src/null.c
	${PROJECT_SOURCE_DIR}/lcc/src/output.c
	${PROJECT_SOURCE_DIR}/lcc/src/prof.c
	${PROJECT_SOURCE_DIR}/lcc/src/profio.c
	${PROJECT_SOURCE_DIR}/lcc/src/simp.c
	${PROJECT_SOURCE_DIR}/lcc/src/stmt.c
	${PROJECT_SOURCE_DIR}/lcc/src/string.c
	${PROJECT_SOURCE_DIR}/lcc/src/sym.c
	${PROJECT_SOURCE_DIR}/lcc/src/symbolic.c
	${PROJECT_SOURCE_DIR}/lcc/src/trace.c
	${PROJECT_SOURCE_DIR}/lcc/src/tree.c
	${PROJECT_SOURCE_DIR}/lcc/src/types.c
)

# compiler
add_executable(q3lcc
	${PROJECT_SOURCE_DIR}/lcc/etc/bytecodelcc.c
	${PROJECT_SOURCE_DIR}/lcc/etc/lcc.c
)

# linker
add_executable(q3asm
	${PROJECT_SOURCE_DIR}/q3asm/q3asm.c
	${PROJECT_SOURCE_DIR}/q3asm/cmdlib.c
)
target_include_directories(q3asm PRIVATE ${PROJECT_SOURCE_DIR}/q3asm)

# example program
add_executable(q3vm ${PROJECT_SOURCE_DIR}/src/main.c)
target_link_libraries(q3vm PRIVATE q3vmlib)

# bytecode example
set(example_sources bg_lib.c g_main.c)
add_custom_target(example ALL
	BYPRODUCTS ${PROJECT_SOURCE_DIR}/example/bg_lib.asm ${PROJECT_SOURCE_DIR}/example/g_main.asm
	COMMAND $<TARGET_FILE:q3lcc> -DQ3_VM -S -Wf-target=bytecode -Wf-g ${example_sources}
	COMMAND_EXPAND_LISTS
	WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/example
	VERBATIM
)
add_custom_command(
	TARGET example
	POST_BUILD
	COMMAND $<TARGET_FILE:q3asm> -f bytecode
	WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/example
	VERBATIM
)
