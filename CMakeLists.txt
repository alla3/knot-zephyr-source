cmake_minimum_required(VERSION 3.8.2)
find_package(Git REQUIRED)
find_program(INSTALL install)

include($ENV{ZEPHYR_BASE}/cmake/app/boilerplate.cmake NO_POLICY_SCOPE)

include(ExternalProject)

zephyr_get_include_directories_for_lang_as_string(C includes)
zephyr_get_system_include_directories_for_lang_as_string(C system_includes)
zephyr_get_compile_options_for_lang_as_string(C c_options)
zephyr_get_compile_definitions_for_lang_as_string(C definitions)

set(PROTO_ROOT ${CMAKE_BINARY_DIR}/external/proto)
set(PROTO_LIB_DIR ${PROTO_ROOT}/bin/lib)
set(PROTO_INCLUDE_DIR ${PROTO_ROOT}/bin/include)

#get_property(RULE_LAUNCH_COMPILE GLOBAL PROPERTY RULE_LAUNCH_COMPILE)

set(configure_flags
  "INSTALL=${INSTALL} -p"
  "CC=${RULE_LAUNCH_COMPILE} ${CMAKE_C_COMPILER}"
  "AR=${CMAKE_AR}"
  "STRIP=${CMAKE_STRIP}"
  "RANLIB=${CMAKE_RANLIB}"
  "CFLAGS=${c_options} ${includes} ${system_includes}"
  #"LDFLAGS=${CMAKE_EXE_LINKER_FLAGS}"
  --host=${CROSS_COMPILE_TARGET}
  --target=${CROSS_COMPILE_TARGET}
  --prefix=${PROTO_ROOT}
  #--build=i586-zephyr-elf
)

ExternalProject_Add(proto
    PREFIX ${PROTO_ROOT}
    BINARY_DIR ${PROTO_ROOT}/src/proto
    SOURCE_DIR ${PROTO_ROOT}/src/proto
    GIT_REPOSITORY https://github.com/CESARBR/knot-protocol-source.git
    CONFIGURE_COMMAND ./bootstrap && ./configure ${configure_flags}

    COMMENT "bootstrapping..." # Text printed when step executes
    BUILD_COMMAND make
)

project(knot_hello_world)
add_dependencies(app proto)
target_sources(app PRIVATE src/main.c)
