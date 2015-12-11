include(CTest)
find_program( MEMORYCHECK_COMMAND valgrind )
find_package(Threads)
set( MEMORYCHECK_COMMAND_OPTIONS "--trace-children=yes --leak-check=full" )

# Set include directories
include_directories(
	../mission_inc
	../platform_inc
	$ENV{CFE_DIR}/fsw/mission_inc
	$ENV{CFE_DIR}/fsw/platform_inc/cpu1
	${CMAKE_CURRENT_SOURCE_DIR}/../../../cfs_lib/fsw/src
	${CMAKE_CURRENT_SOURCE_DIR}
	${CFE_INC_DIRS}
	${OSAL_INC_DIRS}
	${UTF_INCS}
	${APP_INC_DIRS})

add_definitions(-DCFE_LINUX)

set(CMAKE_C_FLAGS "-g -O0 -Wformat=0 -Wno-pointer-to-int-cast -Wno-int-to-pointer-cast -fprofile-arcs -ftest-coverage -m32")

# Add the executables
add_executable(DS-UT 
		${CMAKE_CURRENT_SOURCE_DIR}/ds_utest_app.c
		${CMAKE_CURRENT_SOURCE_DIR}/ds_utest_cmds.c
		${CMAKE_CURRENT_SOURCE_DIR}/ds_utest_file.c
		${CMAKE_CURRENT_SOURCE_DIR}/ds_utest_table.c
		${CMAKE_CURRENT_SOURCE_DIR}/ds_utest.c
		${CMAKE_CURRENT_SOURCE_DIR}/../tables/ds_file_tbl.c
		${CMAKE_CURRENT_SOURCE_DIR}/../tables/ds_filter_tbl.c
		${CMAKE_CURRENT_SOURCE_DIR}/../../../cfs_lib/fsw/src/cfs_utils.c
		${APP_SRC}
		${UTF_SRC})
target_link_libraries(DS-UT -fprofile-arcs gcov ${CMAKE_THREAD_LIBS_INIT})

# We need to both run the test and compare the as-run with the expected output
# with a diff command. 
add_test(NAME ds-ut
    COMMAND ${CMAKE_COMMAND}
    	-DTEST_PROG=$<TARGET_FILE:DS-UT>
    	-DEXPECTED_OUTPUT=${CMAKE_CURRENT_SOURCE_DIR}/ds_utest.out
    	-DACTUAL_OUTPUT=${CMAKE_CURRENT_BINARY_DIR}/ds_utest.out
    -P ${CMAKE_CURRENT_SOURCE_DIR}/runtest.cmake)

# Copy input files to build directory
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/sbsim.in ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
    
# Build tables for the tests
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/disk/TT)
SET(CMAKE_CXX_ARCHIVE_CREATE "<CMAKE_AR> <OBJECTS> ${CMAKE_CURRENT_BINARY_DIR}/disk/TT")
SET(CMAKE_C_ARCHIVE_CREATE "<CMAKE_AR> <OBJECTS> ${CMAKE_CURRENT_BINARY_DIR}/disk/TT")


