##################################################################
#
# Coverage Unit Test build recipe
#
# This CMake file contains the recipe for building the SBN unit tests.
# It is invoked from the parent directory when unit tests are enabled.
#
##################################################################

#
#
# NOTE on the subdirectory structures here:
#
# - "inc" provides local header files shared between the coveragetest,
#    wrappers, and overrides source code units
# - "coveragetest" contains source code for the actual unit test cases
#    The primary objective is to get line/path coverage on the FSW 
#    code units.
# - "wrappers" contains wrappers for the FSW code.  The wrapper adds
#    any UT-specific scaffolding to facilitate the coverage test, and
#    includes the unmodified FSW source file.
#
 
set(UT_NAME sbn)

# Use the UT assert public API, and allow direct
# inclusion of source files that are normally private
include_directories(${PROJECT_SOURCE_DIR}/fsw/src)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/inc)

# Generate a dedicated "testrunner" executable that executes the tests for each FSW code unit
# Although sbn has only one source file, this is done in a loop such that 
# the general pattern should work for several files as well.
set(source_files_under_test)
aux_source_directory("${PROJECT_SOURCE_DIR}/fsw/src" source_files_under_test)
foreach(SRCFILE ${source_files_under_test})
    get_filename_component(UNITNAME "${SRCFILE}" NAME_WE)
    
    set(TESTNAME                "${UT_NAME}-${UNITNAME}")
    set(TESTCASE_SOURCE_FILE    "coveragetest/coveragetest_${UNITNAME}.c")
    
    # build test, but CF includes the .c in the test's source therefore
    # the TESTCASE_SRC is left empty, while the UT_SRC is passed in as the
    # combined test and source (currently done to access 'static' functions)
    # without doing it like this the coverage reports will not be created
    add_cfe_coverage_test("sbn" "${UNITNAME}" "${TESTCASE_SOURCE_FILE}" "${SRCFILE}")
endforeach()
