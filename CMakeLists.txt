# ======================================================================
#  larcv main build file
#
#  cd .../path/to/build/directory
#  source .../path/to/larcv/ups/setup_for_development <-d|-p>
#  cmake [-DCMAKE_INSTALL_PREFIX=/install/path]
#        -DCMAKE_BUILD_TYPE=$CETPKG_TYPE
#        $CETPKG_SOURCE
#  make
#  make test
#  make install
#  make package (builds distribution tarfile)
# ======================================================================


# use cmake 2.8 or later
cmake_minimum_required (VERSION 2.8)

project(larcv)

# cetbuildtools contains our cmake modules
SET ( CETBUILDTOOLS_VERSION $ENV{CETBUILDTOOLS_VERSION} )
IF (NOT CETBUILDTOOLS_VERSION)
    MESSAGE (FATAL_ERROR "ERROR: setup cetbuildtools to get the cmake modules")
ENDIF()

set(CMAKE_MODULE_PATH $ENV{ART_DIR}/Modules
		      $ENV{CETBUILDTOOLS_DIR}/Modules
		      ${CMAKE_MODULE_PATH})

include(CetCMakeEnv)
cet_cmake_env()

cet_set_compiler_flags(DIAGS CAUTIOUS
  WERROR
  NO_UNDEFINED
  EXTRA_FLAGS -pedantic
  EXTRA_CXX_FLAGS -std=c++11 )

cet_report_compiler_flags()

# these are minimum required versions, not the actual product versions
find_ups_product( larlite v1_00_00 )
find_ups_product( cetbuildtools v4_14_01 )

# ADD SOURCE CODE SUBDIRECTORIES HERE
#add_subdirectory(larcv)

execute_process(COMMAND git submodule update --init
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  OUTPUT_VARIABLE submodule_update)

file(GLOB larcv_source "${CMAKE_CURRENT_SOURCE_DIR}/LArCV/*")
file(COPY ${larcv_source}  DESTINATION ${CMAKE_INSTALL_PREFIX}/${flavorqual_dir})

message("Making LArCV...")
set(make_command "source configure.sh &&  make")
execute_process(COMMAND source configure.sh
  WORKING_DIRECTORY ${CMAKE_INSTALL_PREFIX}/${flavorqual_dir}
  OUTPUT_VARIABLE configure_output)
execute_process(COMMAND make
  WORKING_DIRECTORY ${CMAKE_INSTALL_PREFIX}/${flavorqual_dir}
  OUTPUT_VARIABLE make_output)


# tests
add_subdirectory(test)

# ups - table and config files
add_subdirectory(ups)

# packaging utility
include(UseCPack)
