if(BUILD_PERF)
	set(PERF_SRC ${CMAKE_CURRENT_SOURCE_DIR}/perf-main.c)
	if(UNIX AND NOT APPLE)
		list(APPEND PERF_LIB rt)
	endif()

	add_executable(${BINARY_NAME}-perf ${PERF_SRC})
	target_link_libraries(${BINARY_NAME}-perf ${BINARY_NAME} ${PERF_LIB} ${OS_LIB})
	set_target_properties(${BINARY_NAME}-perf PROPERTIES COMPILE_DEFINITIONS "${OS_DEFINES};${FEATURE_DEFINES};${FUNCTION_DEFINES}" RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
	install(TARGETS ${BINARY_NAME}-perf DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT ${BINARY_NAME}-perf)
	install(FILES "${PROJECT_SOURCE_DIR}/tools/perf.py" DESTINATION "${LIBDIR}/${BINARY_NAME}" COMPONENT ${BINARY_NAME}-perf)
endif()

if(BUILD_TEST)
	add_executable(${BINARY_NAME}-fuzz ${CMAKE_CURRENT_SOURCE_DIR}/fuzz-main.c)
	target_link_libraries(${BINARY_NAME}-fuzz ${BINARY_NAME})
	set_target_properties(${BINARY_NAME}-fuzz PROPERTIES COMPILE_DEFINITIONS "${OS_DEFINES};${FEATURE_DEFINES};${FUNCTION_DEFINES}")
	add_executable(tbl-fuzz ${CMAKE_CURRENT_SOURCE_DIR}/tbl-fuzz-main.c)
	target_link_libraries(tbl-fuzz ${BINARY_NAME})
	set_target_properties(tbl-fuzz PROPERTIES COMPILE_DEFINITIONS "${OS_DEFINES};${FEATURE_DEFINES};${FUNCTION_DEFINES}")
	install(TARGETS ${BINARY_NAME}-fuzz tbl-fuzz DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT ${BINARY_NAME}-test)
endif()

if(BUILD_SUITE)
	enable_testing()
	include_directories(AFTER ${CMOCKA_INCLUDE_DIRS})
	link_directories(${CMOCKA_LIBRARY_DIRS})

	foreach(TEST IN LISTS TEST_SRC)
		string(REPLACE "${PROJECT_SOURCE_DIR}/src/" "" TEST_NAME "${TEST}")
		string(REPLACE "/" "-" TEST_NAME "${TEST_NAME}")
		string(REPLACE "-test" "" TEST_NAME "${TEST_NAME}")
		string(REPLACE ".c" "" TEST_NAME "${TEST_NAME}")
		add_executable(test-${TEST_NAME} ${TEST})
		target_link_libraries(test-${TEST_NAME} ${BINARY_NAME} ${PLATFORM_LIBRARY} cmocka)
		set_target_properties(test-${TEST_NAME} PROPERTIES COMPILE_DEFINITIONS "${OS_DEFINES};${FEATURE_DEFINES};${FUNCTION_DEFINES}")
		add_test(${TEST_NAME} test-${TEST_NAME})
	endforeach()
endif()

if(BUILD_CINEMA)
	enable_testing()
	add_executable(${BINARY_NAME}-cinema ${CMAKE_CURRENT_SOURCE_DIR}/cinema-main.c)
	target_link_libraries(${BINARY_NAME}-cinema ${BINARY_NAME} ${PLATFORM_LIBRARY})
	set_target_properties(${BINARY_NAME}-cinema PROPERTIES COMPILE_DEFINITIONS "${OS_DEFINES};${FEATURE_DEFINES};${FUNCTION_DEFINES}")
	add_test(cinema ${BINARY_NAME}-cinema -v)
	install(TARGETS ${BINARY_NAME}-cinema DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT ${BINARY_NAME}-test)
endif()
