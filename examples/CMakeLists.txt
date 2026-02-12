# Dependencies
find_package("OpenSSL" REQUIRED COMPONENTS "Crypto" "SSL")

# Makes a new target for each example file.
function(make_example example_file)
    message("${example_file}") # TODO
    # Target
    get_filename_component(EXAMPLE_TARGET "${example_file}" NAME_WE)
    add_executable("${EXAMPLE_TARGET}" "${example_file}")

    # Headers
    target_include_directories("${EXAMPLE_TARGET}" PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/>
                                                           $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
    target_include_directories("${EXAMPLE_TARGET}" PRIVATE $<BUILD_INTERFACE:${PROJECT_CURRENT_SOURCE_DIR}/include/>
                                                           $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

    # Link libraries
    target_link_libraries("${EXAMPLE_TARGET}" PRIVATE "Boost::date_time" "Boost::regex")
    target_link_libraries("${EXAMPLE_TARGET}" PRIVATE "mailio")
    target_link_libraries("${EXAMPLE_TARGET}" PRIVATE "OpenSSL::Crypto" "OpenSSL::SSL")
endfunction()

file(GLOB EXAMPLE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")

foreach(file ${EXAMPLE_FILES})
    make_example("${file}")
endforeach()
