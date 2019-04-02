
enable_language(C)

include_directories(../../third_party)

add_definitions(-DUS_RCC_EXECUTABLE_NAME=\"${US_RCC_EXECUTABLE_OUTPUT_NAME}\")

set(srcs ResourceCompiler.cpp ../../third_party/miniz.c ../../third_party/jsoncpp.cpp)

add_executable(${US_RCC_EXECUTABLE_TARGET} ${srcs})
set_property(TARGET ${US_RCC_EXECUTABLE_TARGET} PROPERTY OUTPUT_NAME ${US_RCC_EXECUTABLE_OUTPUT_NAME})
if(WIN32)
    target_link_libraries(${US_RCC_EXECUTABLE_TARGET} Shlwapi)
endif()

set_property(TARGET ${US_RCC_EXECUTABLE_TARGET} APPEND PROPERTY
             COMPILE_DEFINITIONS "MINIZ_NO_ARCHIVE_READING_API;MINIZ_NO_ZLIB_COMPATIBLE_NAMES")

set_property(TARGET ${US_RCC_EXECUTABLE_TARGET}
  APPEND PROPERTY
  RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/tools/${PROJECT_NAME_LOWER}")

file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/tools/${PROJECT_NAME_LOWER}")
file(INSTALL 
  "${CMAKE_BINARY_DIR}/bin/${US_RCC_EXECUTABLE_TARGET}${US_GLOBAL_VERSION_SUFFIX}.exe"
  DESTINATION "${CMAKE_BINARY_DIR}/tools/${PROJECT_NAME_LOWER}")
# set_property(TARGET ${US_RCC_EXECUTABLE_TARGET} APPEND PROPERTY IMPORTED_CONFIGURATIONS DEBUG)
# set_target_properties(${US_RCC_EXECUTABLE_TARGET} PROPERTIES
#   IMPORTED_LOCATION_DEBUG "${CMAKE_CURRENT_BINARY_DIR}/${EXES_COMPILATION_PATH}${US_RCC_EXECUTABLE_TARGET}${US_GLOBAL_VERSION_SUFFIX}.exe")

message(STATUS "US_SDK_INSTALL_COMPONENT = ${US_SDK_INSTALL_COMPONENT}")
if(NOT US_NO_INSTALL)
    install(TARGETS ${US_RCC_EXECUTABLE_TARGET}
            EXPORT ${PROJECT_NAME}Targets
            FRAMEWORK DESTINATION . ${US_SDK_INSTALL_COMPONENT}
            RUNTIME DESTINATION "tools/cppmicroservices" ${US_SDK_INSTALL_COMPONENT})
endif()

if(CMAKE_CROSSCOMPILING)
  # When cross-compiling, import the host machine executable from a file
  # See https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/CrossCompiling
  include(${IMPORT_EXECUTABLES})
else()
  # Export the host usResourceCompiler target to a file so it can be imported when cross-compiling.
  # Use the NAMESPACE option of EXPORT() to get a different target name for ${US_RCC_EXECUTABLE_TARGET}
  # when exporting.
  # Doing this allows the host built usResourceCompiler to be used when cross-compiling and allows
  # the usResourceCompiler to be built and installed for the target machine.
  export( TARGETS ${US_RCC_EXECUTABLE_TARGET} FILE ${IMPORT_EXECUTABLES} NAMESPACE native-)
endif()
