project(modbus)

include(ExternalProject)
set(MODBUS_DIR ${PROJECT_SOURCE_DIR})
set(MODBUS_BIN ${CMAKE_BINARY_DIR}/thirdparty/libmodbus)
set(MODBUS_STATIC_LIB ${MODBUS_BIN}/lib/libmodbus.a)
set(MODBUS_INCLUDES ${MODBUS_BIN}/include)

file(MAKE_DIRECTORY ${MODBUS_INCLUDES})

ExternalProject_Add(
  libmodbus
  PREFIX ${MODBUS_BIN}
  SOURCE_DIR ${MODBUS_DIR}
  DOWNLOAD_COMMAND cd ${MODBUS_DIR} && git clean -dfX && ${MODBUS_DIR}/autogen.sh
  CONFIGURE_COMMAND ${MODBUS_DIR}/configure --host=${CONFIG_HOST} --target=${CONFIG_HOST} --srcdir=${MODBUS_DIR} --prefix=${MODBUS_BIN} --enable-static=yes && cp ${MODBUS_BIN}/src/libmodbus-build/config.h ${MODBUS_INCLUDES}
  BUILD_COMMAND make -j8
  BUILD_BYPRODUCTS ${MODBUS_STATIC_LIB}
  )

add_library(modbus STATIC IMPORTED GLOBAL)
add_dependencies(modbus libmodbus)

set_target_properties(modbus PROPERTIES IMPORTED_LOCATION ${MODBUS_STATIC_LIB})
set_target_properties(modbus PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${MODBUS_INCLUDES})

file( COPY ${MODBUS_DIR}/src/modbus-private.h DESTINATION ${MODBUS_INCLUDES}/modbus )
file( COPY ${MODBUS_DIR}/src/modbus-rtu-private.h DESTINATION ${MODBUS_INCLUDES}/modbus )
file( COPY ${MODBUS_DIR}/src/modbus-tcp-private.h DESTINATION ${MODBUS_INCLUDES}/modbus )
file( COPY ${MODBUS_BIN}/lib DESTINATION ${CMAKE_BINARY_DIR} )
