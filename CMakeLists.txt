cmake_minimum_required(VERSION "3.30.0")

if("${CMAKE_CURRENT_BINARY_DIR}/" STREQUAL "${CMAKE_CURRENT_SOURCE_DIR}/")
    message(FATAL_ERROR "In-source builds are not allowed.")
endif()

# Modules
include("CMakeDependentOption")
include("CMakePackageConfigHelpers")
include("GenerateExportHeader")
include("GNUInstallDirs")

# Options
option(BUILD_SHARED_LIBS "Turn on to build mailio as a shared library. When off mailio is build as a static library."
       ON)
option(MAILIO_BUILD_DOCUMENTATION "Turn on to build doxygen based documentation." ON)
option(MAILIO_BUILD_EXAMPLES "Turn on to build examples." ON)
option(MAILIO_BUILD_TESTS "Turn on to build the tests." ON)

cmake_dependent_option(MAILIO_BUILD_LATEX_DOCUMENTATION "Build latex documentation" ON MAILIO_BUILD_DOCUMENTATION ON)
cmake_dependent_option(MAILIO_DYN_LINK_TESTS "Turn on to dynamically link the tests." ON MAILIO_BUILD_TESTS ON)

# General
project(
    "mailio"
    DESCRIPTION "Cross-platform MIME library for SMTP, POP3, and IMAP protocols."
    HOMEPAGE_URL "https://github.com/karastojko/mailio"
    LANGUAGES "CXX"
    VERSION "0.26.0")
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Dependencies
if(${WIN32})
    set(Boost_USE_STATIC_LIBS ON)
endif()

find_package("Boost" REQUIRED COMPONENTS "date_time" "regex")

if(${MAILIO_BUILD_DOCUMENTATION})
    find_package(
        "Doxygen" REQUIRED
        COMPONENTS "doxygen"
        OPTIONAL_COMPONENTS "dot")
endif()

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package("Threads" REQUIRED)

# Target
set(PROJECT_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/base64.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/binary.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/bit7.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/bit8.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/codec.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/dialog.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/imap.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/mailboxes.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/message.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/mime.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/percent.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/pop3.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/quoted_printable.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/q_codec.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/smtp.cpp")

add_library("${PROJECT_NAME}" "${PROJECT_SOURCES}")

# Headers
target_include_directories("${PROJECT_NAME}" PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/>
                                                    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/>)
target_include_directories("${PROJECT_NAME}" PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
                                                    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/>)

# Link libraries
target_link_libraries("${PROJECT_NAME}" PUBLIC "Boost::date_time" "Boost::regex")

if(${MINGW})
    target_link_libraries("${PROJECT_NAME}" PUBLIC "-lws2_32")
endif()

# Compile options
if(${MSVC})
    target_compile_options("${PROJECT_NAME}" PRIVATE "/W2" "/WX")
else()
    target_compile_options("${PROJECT_NAME}" PRIVATE "-Wall" "-Wextra" "-Wpedantic")
endif()

# Documentation
if(${MAILIO_BUILD_DOCUMENTATION})
    set(DOXYGEN_GENERATE_LATEX ${MAILIO_BUILD_LATEX_DOCUMENTATION})
    set(DOXYGEN_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/docs/${PROJECT_NAME}/")

    doxygen_add_docs("${PROJECT_NAME}-documentation" "${CMAKE_CURRENT_SOURCE_DIR}/include/" ALL
                     WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/")
endif()

# Tests
if(${MAILIO_BUILD_TESTS})
    enable_testing()
    add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/test/")
endif()

# Examples
if(${MAILIO_BUILD_EXAMPLES})
    add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/examples/")
endif()

# Installation
generate_export_header("${PROJECT_NAME}" EXPORT_FILE_NAME "${CMAKE_CURRENT_BINARY_DIR}/export.hpp")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.pc.in" "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc"
               @ONLY)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/include/version.hpp.in" "${CMAKE_CURRENT_BINARY_DIR}/version.hpp")

install(
    TARGETS "${PROJECT_NAME}"
    EXPORT "${PROJECT_NAME}-export"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}/"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}/"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}/"
    INCLUDES
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/")
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}/" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc" DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig/")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/export.hpp" DESTINATION "${INCLUDE_INSTALL_DIR}/${PROJECT_NAME}/")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/version.hpp" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/")

# Target installation
install(
    EXPORT "${PROJECT_NAME}-export"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}/"
    FILE "${PROJECT_NAME}-targets.cmake"
    NAMESPACE "${PROJECT_NAME}::")
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}-config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}/")
write_basic_package_version_file(
    "${PROJECT_NAME}-config-version.cmake"
    COMPATIBILITY SameMajorVersion
    VERSION "${PROJECT_VERSION}")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}/")
