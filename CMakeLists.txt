cmake_minimum_required(VERSION 2.8)
project(X-IMU-API)

enable_testing()

set(X-IMU-API_MAJOR_VERION 0)
set(X-IMU-API_MINOR_VERION 1)
set(X-IMU-API_PATCH_VERION 0)

# set compiler flags
set(CMAKE_CXX_FLAGS "${CAMKE_CXX_FLAGS} -std=c++11")

# set main include directory
include_directories(
  include # main include directory
)
# add a target to generate API documentation with Doxygen
find_package(Doxygen)
if(DOXYGEN_FOUND)
	set(doxyfile_in ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
    set(doxyfile ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

    configure_file(${doxyfile_in} ${doxyfile} @ONLY)

    add_custom_target(doc
        COMMAND ${DOXYGEN_EXECUTABLE} ${doxyfile}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM)

    install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/html DESTINATION share/doc)
endif(DOXYGEN_FOUND)



# add functionality to the library
add_library(X-IMU-API STATIC
  src/ximuapi/packet/packet_encoding.cpp
  src/ximuapi/utils/fixed_float.cpp
  src/ximuapi/data/register_data.cpp
  src/ximuapi/data/datetime_data.cpp
  src/ximuapi/data/digital_port_bits.cpp
  src/ximuapi/data/digital_io_data.cpp
  src/ximuapi/data/cal_inertial_and_magnetic_data.cpp
  src/ximuapi/data/raw_inertial_and_magnetic_data.cpp
  src/ximuapi/data/raw_battery_and_thermometer_data.cpp
  src/ximuapi/data/cal_battery_and_thermometer_data.cpp
  src/ximuapi/serialization/reader_base.cpp
  src/ximuapi/serialization/packet_reader.cpp
  src/ximuapi/serialization/writer_base.cpp
)

# setup testing

# add c++ program
add_executable (PKT_ENC_TEST tests/packet_encoding_test.cpp)
add_executable (PKT_CNST_TEST tests/packet_construction_test.cpp)
add_executable (FXD_FLT_TEST tests/fixed_float_test.cpp)
add_executable (DATA_TYPES_TEST tests/data_types_test.cpp)
add_executable (DATA_SERIZ_READ tests/data_serialization_test.cpp)
add_executable (DATA_SERIZ_WRITE tests/data_serial_writer_test.cpp)


# link the x-IMU-API with the programs
target_link_libraries(PKT_ENC_TEST X-IMU-API)
target_link_libraries(PKT_CNST_TEST X-IMU-API)
target_link_libraries(FXD_FLT_TEST X-IMU-API)
target_link_libraries(DATA_TYPES_TEST X-IMU-API)
target_link_libraries(DATA_SERIZ_READ X-IMU-API)
target_link_libraries(DATA_SERIZ_WRITE X-IMU-API)

# add ximu port dump for as test data
configure_file(data/ximu.dmp ximu.dmp COPYONLY)

# perform the test
add_test(PACKET_ENCODING_DECODING PKT_ENC_TEST)
add_test(PACKET_CONSTRUCTION PKT_CNST_TEST)
add_test(FIXED_FLOAT FXD_FLT_TEST)
add_test(DATA_TYPES DATA_TYPES_TEST)
add_test(DATA_DESERIALIZER_RECIEVER DATA_SERIZ_READ)
add_test(DATA_DESERIALIZER_SENDER DATA_SERIZ_WRITE)