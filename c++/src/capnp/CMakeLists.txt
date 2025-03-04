
# capnp ========================================================================

set(capnp_sources_lite
  c++.capnp.c++
  blob.c++
  arena.c++
  layout.c++
  list.c++
  any.c++
  message.c++
  schema.capnp.c++
  stream.capnp.c++
  serialize.c++
  serialize-packed.c++
)
set(capnp_sources_heavy
  schema.c++
  schema-loader.c++
  dynamic.c++
  stringify.c++
)
if(NOT CAPNP_LITE)
  set(capnp_sources ${capnp_sources_lite} ${capnp_sources_heavy})
else()
  set(capnp_sources ${capnp_sources_lite})
endif()

set(capnp_headers
  c++.capnp.h
  common.h
  blob.h
  endian.h
  layout.h
  orphan.h
  list.h
  any.h
  message.h
  capability.h
  membrane.h
  dynamic.h
  schema.h
  schema.capnp.h
  stream.capnp.h
  schema-lite.h
  schema-loader.h
  schema-parser.h
  pretty-print.h
  serialize.h
  serialize-async.h
  serialize-packed.h
  serialize-text.h
  pointer-helpers.h
  generated-header-support.h
  raw-schema.h
)
set(capnp_compat_headers
  compat/std-iterator.h
)
set(capnp_schemas
  c++.capnp
  schema.capnp
  stream.capnp
)
add_library(capnp ${capnp_sources})
add_library(CapnProto::capnp ALIAS capnp)
target_link_libraries(capnp PUBLIC kj)
#make sure external consumers don't need to manually set the include dirs
get_filename_component(PARENT_DIR ${CMAKE_CURRENT_SOURCE_DIR} DIRECTORY)
target_include_directories(capnp INTERFACE
  $<BUILD_INTERFACE:${PARENT_DIR}>
  $<INSTALL_INTERFACE:include>
)
# Ensure the library has a version set to match autotools build
set_target_properties(capnp PROPERTIES VERSION ${VERSION})
install(TARGETS capnp ${INSTALL_TARGETS_DEFAULT_ARGS})
install(FILES ${capnp_headers} ${capnp_schemas} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp")
install(FILES ${capnp_compat_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp/compat")

set(capnp-rpc_sources
  serialize-async.c++
  capability.c++
  membrane.c++
  dynamic-capability.c++
  rpc.c++
  rpc.capnp.c++
  rpc-twoparty.c++
  rpc-twoparty.capnp.c++
  persistent.capnp.c++
  ez-rpc.c++
)
set(capnp-rpc_headers
  rpc-prelude.h
  rpc.h
  rpc-twoparty.h
  rpc.capnp.h
  rpc-twoparty.capnp.h
  persistent.capnp.h
  ez-rpc.h
)
set(capnp-rpc_schemas
  rpc.capnp
  rpc-twoparty.capnp
  persistent.capnp
)
if(NOT CAPNP_LITE)
  add_library(capnp-rpc ${capnp-rpc_sources})
  add_library(CapnProto::capnp-rpc ALIAS capnp-rpc)
  target_link_libraries(capnp-rpc PUBLIC capnp kj-async kj)
  # Ensure the library has a version set to match autotools build
  set_target_properties(capnp-rpc PROPERTIES VERSION ${VERSION})
  install(TARGETS capnp-rpc ${INSTALL_TARGETS_DEFAULT_ARGS})
  install(FILES ${capnp-rpc_headers} ${capnp-rpc_schemas} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp")
endif()

# capnp-json ========================================================================

set(capnp-json_sources
  compat/json.c++
  compat/json.capnp.c++
)
set(capnp-json_headers
  compat/json.h
  compat/json.capnp.h
)
set(capnp-json_schemas
  compat/json.capnp
)
if(NOT CAPNP_LITE)
  add_library(capnp-json ${capnp-json_sources})
  add_library(CapnProto::capnp-json ALIAS capnp-json)
  target_link_libraries(capnp-json PUBLIC capnp kj-async kj)
  # Ensure the library has a version set to match autotools build
  set_target_properties(capnp-json PROPERTIES VERSION ${VERSION})
  install(TARGETS capnp-json ${INSTALL_TARGETS_DEFAULT_ARGS})
  install(FILES ${capnp-json_headers} ${capnp-json_schemas} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp/compat")
endif()

# capnp-websocket ========================================================================

set(capnp-websocket_sources
  compat/websocket-rpc.c++
)
set(capnp-websocket_headers
  compat/websocket-rpc.h
)
if(NOT CAPNP_LITE)
  add_library(capnp-websocket ${capnp-websocket_sources})
  add_library(CapnProto::capnp-websocket ALIAS capnp-websocket)
  target_link_libraries(capnp-websocket PUBLIC capnp capnp-rpc kj-http kj-async kj)
  # Ensure the library has a version set to match autotools build
  set_target_properties(capnp-websocket PROPERTIES VERSION ${VERSION})
  install(TARGETS capnp-websocket ${INSTALL_TARGETS_DEFAULT_ARGS})
  install(FILES ${capnp-websocket_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp/compat")
endif()

# Tools/Compilers ==============================================================

set(capnpc_sources
  compiler/type-id.c++
  compiler/error-reporter.c++
  compiler/lexer.capnp.c++
  compiler/lexer.c++
  compiler/grammar.capnp.c++
  compiler/parser.c++
  compiler/generics.c++
  compiler/node-translator.c++
  compiler/compiler.c++
  schema-parser.c++
  serialize-text.c++
)
if(NOT CAPNP_LITE)
  add_library(capnpc ${capnpc_sources})
  target_link_libraries(capnpc PUBLIC capnp kj)
  # Ensure the library has a version set to match autotools build
  set_target_properties(capnpc PROPERTIES VERSION ${VERSION})
  install(TARGETS capnpc ${INSTALL_TARGETS_DEFAULT_ARGS})
  install(FILES ${capnpc_headers} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp")
endif()

if(NOT CAPNP_LITE)
  add_executable(capnp_tool
    compiler/module-loader.c++
    compiler/capnp.c++
  )
  target_link_libraries(capnp_tool capnpc capnp-json capnp kj)
  set_target_properties(capnp_tool PROPERTIES OUTPUT_NAME capnp)
  set_target_properties(capnp_tool PROPERTIES CAPNP_INCLUDE_DIRECTORY
    $<JOIN:$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>,$<INSTALL_INTERFACE:${CMAKE_INSTALL_BINDIR}/..>>
  )
  target_compile_definitions(capnp_tool PRIVATE
    "CAPNP_INCLUDE_DIR=\"${CMAKE_INSTALL_FULL_INCLUDEDIR}\""
    "VERSION=\"${VERSION}\""
  )

  add_executable(capnpc_cpp
    compiler/capnpc-c++.c++
  )
  target_link_libraries(capnpc_cpp capnp kj)
  set_target_properties(capnpc_cpp PROPERTIES OUTPUT_NAME capnpc-c++)
  #Capnp tool needs capnpc_cpp location. But cmake deprecated LOCATION property.
  #So we use custom property to pass location
  set_target_properties(capnpc_cpp PROPERTIES CAPNPC_CXX_EXECUTABLE
    $<TARGET_FILE:capnpc_cpp>
  )

  add_executable(capnpc_capnp
    compiler/capnpc-capnp.c++
  )
  target_link_libraries(capnpc_capnp capnp kj)
  set_target_properties(capnpc_capnp PROPERTIES OUTPUT_NAME capnpc-capnp)

  install(TARGETS capnp_tool capnpc_cpp capnpc_capnp ${INSTALL_TARGETS_DEFAULT_ARGS})

  if(WIN32)
    # On Windows platforms symlinks are not guaranteed to support. Also different version of CMake handle create_symlink in a different way.
    # The most portable way in this case just copy the file.
    install(CODE "execute_process(COMMAND \"${CMAKE_COMMAND}\" -E copy \"\$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/capnp${CMAKE_EXECUTABLE_SUFFIX}\" \"\$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/capnpc${CMAKE_EXECUTABLE_SUFFIX}\")")
  else()
    # Symlink capnpc -> capnp
    install(CODE "execute_process(COMMAND \"${CMAKE_COMMAND}\" -E create_symlink capnp${CMAKE_EXECUTABLE_SUFFIX} \"\$ENV{DESTDIR}${CMAKE_INSTALL_FULL_BINDIR}/capnpc${CMAKE_EXECUTABLE_SUFFIX}\")")
  endif()
endif()  # NOT CAPNP_LITE

# http-over-capnp ==============================================================

# This is put here, after the Tools/Compilers section because we need the capnp tool to generate the sources from the schema

set(http-over-capnp_sources
	compat/byte-stream.c++
	compat/http-over-capnp.c++
)
set(http-over-capnp_headers
	compat/byte-stream.h
	compat/http-over-capnp.h 
)
set(http-over-capnp_schemas
	compat/byte-stream.capnp
	compat/http-over-capnp.capnp 
)

if(NOT CAPNP_LITE)
	capnp_generate_cpp(http-over-capnp_sources http-over-capnp_headers ${http-over-capnp_schemas})

	add_library(http-over-capnp ${http-over-capnp_sources})
	add_library(CapnProto::http-over-capnp ALIAS http-over-capnp)
	target_link_libraries(http-over-capnp PUBLIC capnp kj-http)
	set_target_properties(http-over-capnp PROPERTIES VERSION ${VERSION})
	install(TARGETS http-over-capnp ${INSTALL_TARGETS_DEFAULT_ARGS})
	install(FILES ${http-over-capnp_headers} ${http-over-capnp_schemas} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/capnp/compat")
endif()
# Tests ========================================================================

if(BUILD_TESTING)
  set(test_capnp_files
    test.capnp
    test-import.capnp
    test-import2.capnp
    compat/json-test.capnp
  )

  set(CAPNPC_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/test_capnp")
  include_directories("${CAPNPC_OUTPUT_DIR}")
  file(MAKE_DIRECTORY "${CAPNPC_OUTPUT_DIR}")
  # Tell capnp_generate_cpp to set --src-prefix to our parent directory. This allows us to pass our
  # .capnp files relative to this directory, but have their canonical name end up as
  # capnp/test.capnp, capnp/test-import.capnp, etc.
  get_filename_component(CAPNPC_SRC_PREFIX "${CMAKE_CURRENT_SOURCE_DIR}" DIRECTORY)

  capnp_generate_cpp(test_capnp_cpp_files test_capnp_h_files ${test_capnp_files})

  # TODO(cleanup): capnp-tests and capnp-heavy-tests both depend on the test.capnp output files. In
  #   a parallel Makefile-based build (maybe others?), they can race and cause the custom capnp
  #   command in capnp_generate_cpp() to run twice. To get around this I'm using a custom target to
  #   force CMake to generate race-free Makefiles. Remove this garbage when we move to a
  #   target-based capnp_generate() command, as that will make CMake do the right thing by default.
  add_custom_target(test_capnp DEPENDS ${test_capnp_cpp_files} ${test_capnp_h_files})

  if(CAPNP_LITE)
    set(test_libraries capnp kj-test kj)
  else()
    set(test_libraries capnp-json capnp-rpc capnp-websocket capnp capnpc kj-http kj-async kj-test kj)
  endif()

  add_executable(capnp-tests
    common-test.c++
    blob-test.c++
    endian-test.c++
    endian-fallback-test.c++
    layout-test.c++
    any-test.c++
    message-test.c++
    encoding-test.c++
    orphan-test.c++
    serialize-test.c++
    serialize-packed-test.c++
    canonicalize-test.c++
    fuzz-test.c++
    test-util.c++
    ${test_capnp_cpp_files}
    ${test_capnp_h_files}
  )
  target_link_libraries(capnp-tests ${test_libraries})
  add_dependencies(capnp-tests test_capnp)
  add_dependencies(check capnp-tests)
  add_test(NAME capnp-tests-run COMMAND capnp-tests)

  if(NOT CAPNP_LITE)
    add_executable(capnp-heavy-tests
      endian-reverse-test.c++
      capability-test.c++
      membrane-test.c++
      schema-test.c++
      schema-loader-test.c++
      schema-parser-test.c++
      dynamic-test.c++
      stringify-test.c++
      serialize-async-test.c++
      serialize-text-test.c++
      rpc-test.c++
      rpc-twoparty-test.c++
      ez-rpc-test.c++
      compiler/lexer-test.c++
      compiler/type-id-test.c++
      test-util.c++
      compat/json-test.c++
      compat/websocket-rpc-test.c++
      ${test_capnp_cpp_files}
      ${test_capnp_h_files}
    )
    target_link_libraries(capnp-heavy-tests ${test_libraries})
    if(NOT MSVC)
      set_target_properties(capnp-heavy-tests
        PROPERTIES COMPILE_FLAGS "-Wno-deprecated-declarations"
      )
    endif()

    add_dependencies(capnp-heavy-tests test_capnp)
    add_dependencies(check capnp-heavy-tests)
    add_test(NAME capnp-heavy-tests-run COMMAND capnp-heavy-tests)

    add_executable(capnp-evolution-tests compiler/evolution-test.c++)
    target_link_libraries(capnp-evolution-tests capnpc capnp kj)
    add_dependencies(check capnp-evolution-tests)
    add_test(NAME capnp-evolution-tests-run COMMAND capnp-evolution-tests)
  endif()  # NOT CAPNP_LITE
endif()  # BUILD_TESTING

if(DEFINED ENV{LIB_FUZZING_ENGINE})
  add_executable(capnp_llvm_fuzzer_testcase llvm-fuzzer-testcase.c++ test-util.c++ test-util.h ${test_capnp_cpp_files} ${test_capnp_h_files})
  target_link_libraries(capnp_llvm_fuzzer_testcase capnp-rpc capnp kj kj-async capnp-json $ENV{LIB_FUZZING_ENGINE})
endif()
