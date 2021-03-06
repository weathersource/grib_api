# tests/CMakeLists.txt
#
# Configure the file which all CMake tests will include
configure_file( include.ctest.sh.in  include.ctest.sh @ONLY )

execute_process( COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/include.sh ${CMAKE_CURRENT_BINARY_DIR} )
execute_process( COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/utils.sh   ${CMAKE_CURRENT_BINARY_DIR} )

# Build the executables used by test scripts
################################################
list( APPEND test_bins
      read_any
      julian
      index
      multi_from_message
      read_index
      unit_tests
      gauss_sub
      grib_util_set_spec
)

foreach( tool ${test_bins} )
    # here we use the fact that each tool has only one C file that matches its name
    ecbuild_add_executable( TARGET    ${tool}
                            NOINSTALL
                            SOURCES   ${tool}.c
                            LIBS      grib_api
    )
endforeach()

# Now add each test (shell scripts)
#################################################
list( APPEND tests1
    definitions
    calendar
    unit_tests
    md5
    uerra
)
list( APPEND tests2
    ieee
    grib1to2
    grib2to1
    badgrib
    ls
    filter
    multi
    budg
    gridType
	octahedral
    global
    concept
    decimalPrecision
    bitsPerValue
    get_fail
    missing
    local
    step
    set
    iterator
    compare
    level
    index
    bitmap
    list
    second_order
    multi_from_message
    change_scanning
    julian
    statistics
    tigge
    tigge_conversions
    read_any
    padding
    debug
    grib_util_set_spec
    neg_fctime
)

# These tests do not require any data downloads
foreach( test ${tests1} )
    ecbuild_add_test( TARGET grib_api_t_${test}
                  TYPE SCRIPT
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/${test}.sh
    )
endforeach()
ecbuild_add_test( TARGET grib_api_t_check_gaussian_grids
                  TYPE SCRIPT
                  CONDITION  GRIB_API_INSTALL_EXTRA_TOOLS
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/check_gaussian_grids.sh
)


# Note: making the test dependent on the grib files (with DEPENDS)
#       means they will be downloaded at "make" time
#       rather than when you do "make test". Use TEST_DEPENDS instead
foreach( test ${tests2} )
    ecbuild_add_test( TARGET grib_api_t_${test}
                  TYPE SCRIPT
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/${test}.sh
                  TEST_DEPENDS grib_api_download_gribs grib_api_download_tigge_gribs
    )
endforeach()

###########################################
# Note: the reference file is in the tests dir not data dir!
ecbuild_add_test( TARGET grib_api_t_lamb_az_eq_area
                  TYPE SCRIPT
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/lamb_az_eq_area.sh
                  RESOURCES lamb_az_eq_area.ref
                  TEST_DEPENDS grib_api_download_gribs
)

# These ones are conditional
ecbuild_add_test( TARGET grib_api_t_grib_to_netcdf
                  TYPE SCRIPT
                  CONDITION HAVE_NETCDF
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/grib_to_netcdf.sh
                  TEST_DEPENDS grib_api_download_gribs
)

ecbuild_add_test( TARGET grib_api_t_jpeg
                  TYPE SCRIPT
                  CONDITION HAVE_JPEG
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/jpeg.sh
                  TEST_DEPENDS grib_api_download_gribs
)

ecbuild_add_test( TARGET grib_api_t_ccsds
                  TYPE SCRIPT
                  CONDITION HAVE_AEC
                  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/ccsds.sh
                  TEST_DEPENDS grib_api_download_gribs
)
