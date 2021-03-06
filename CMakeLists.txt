project(LPMP)

cmake_minimum_required(VERSION 2.8.12)
# cmake_policy(SET CMP0074 NEW)

# C++17
set(CMAKE_CXX_FLAGS "-std=c++17")
#set(CMAKE_CXX_FLAGS "-fuse-ld=gold")

# compiler options
add_definitions(-march=native)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

#linker options
#set(CMAKE_SHARED_LINKER_FLAGS "--compress-debug-sections=zlib")
#add_link_options("--compress-debug-sections=zlib")

# information for e.g. YouCompleteMe
SET( CMAKE_EXPORT_COMPILE_COMMANDS ON )
IF( EXISTS "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json" )
    EXECUTE_PROCESS( COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json
        ${CMAKE_CURRENT_SOURCE_DIR}/compile_commands.json
        )
ENDIF()

add_library(LPMP INTERFACE)
target_include_directories(LPMP INTERFACE external)
target_include_directories(LPMP INTERFACE external/qpbo/include)
target_include_directories(LPMP INTERFACE external/maxflow/include)
target_include_directories(LPMP INTERFACE include)
target_include_directories(LPMP INTERFACE external/PEGTL)
target_include_directories(LPMP INTERFACE external/TCLAP/include)
target_include_directories(LPMP INTERFACE external/libsimdpp)
target_include_directories(LPMP INTERFACE external/meta/include)
target_include_directories(LPMP INTERFACE external/BCFW-Bundle/include)
target_include_directories(LPMP INTERFACE external/ConicBundle/include)
target_include_directories(LPMP INTERFACE external/arboricity)
target_include_directories(LPMP INTERFACE external/MCF-SSP)
target_include_directories(LPMP INTERFACE external/andres-graph/include)
target_include_directories(LPMP INTERFACE external/tsl-robin-map/include)
target_include_directories(LPMP INTERFACE external/cpp-taskflow) 
target_include_directories(LPMP INTERFACE external/pybind11)
add_custom_target(
    Cudd_lib
    COMMAND ""#"autoreconf -f -i && ./configure --enable-obj --enable-shared && make"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/external/cudd/"
    )
add_library(Cudd INTERFACE)
add_dependencies(Cudd Cudd_lib)
target_link_libraries(Cudd INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/external/cudd/cudd/.libs/libcudd.a")
#ExternalProject_Add(Cudd_project
#    SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/external/cudd/"
#    PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/external/cudd/
#    CONFIGURE_COMMAND "./configure --enable-obj"
#    BUILD_COMMAND "make"
#    )
#    -DCMAKE_INSTALL_PREFIX=${CMAKE_SOURCE_DIR}/external/cudd
#    -DCMAKE_Fortran_COMPILER=${CMAKE_Fortran_COMPILER}
#    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
#    -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
#    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#add_library(Cudd STATIC IMPORTED)
#set_property(TARGET Cudd PROPERTY IMPORTED_LOCATION ${install_dir}/lib/libluajit-5.1.a)
#add_dependencies(Cudd Cudd_project)
target_include_directories(LPMP INTERFACE external/cudd/cudd)
target_include_directories(LPMP INTERFACE external/cudd/cplusplus) 
target_link_libraries(LPMP INTERFACE pthread)

# for opengm #
target_include_directories(LPMP INTERFACE external/opengm/include)

find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()
target_link_libraries(LPMP INTERFACE OpenMP::OpenMP_CXX)

find_package(HDF5 REQUIRED CXX)
add_library(HDF5 INTERFACE)
target_include_directories(HDF5 INTERFACE ${HDF5_INCLUDE_DIR})
target_link_libraries(HDF5 INTERFACE ${HDF5_LIBRARIES})
target_link_libraries(HDF5 INTERFACE ${HDF5_CXX_LIBRARIES})
target_compile_definitions(HDF5 INTERFACE ${HDF5_DEFINITIONS})
message(STATUS ${HDF5_LIBRARIES})
message(STATUS ${HDF5_INCLUDE_DIR})
##############

enable_testing()

add_subdirectory(external/qpbo)
add_subdirectory(external/BCFW-Bundle) 
add_subdirectory(external/DD_ILP)
add_subdirectory(external/ConicBundle)
add_subdirectory(external/arboricity)
set(PYBIND11_CPP_STANDARD -std=c++17)
set(PYBIND11_INSTALL ON CACHE BOOL "enable pybind11 bindings.")
add_subdirectory(external/pybind11)
target_include_directories(LPMP INTERFACE external/Eigen)

target_link_libraries(LPMP INTERFACE DD_ILP)

add_subdirectory(src)
enable_testing()
add_subdirectory(test)
add_subdirectory(doc)

option(BUILD_DOC "Build documentation" OFF)
if(BUILD_DOC)
    find_package(Doxygen REQUIRED)
configure_file(Doxyfile.in Doxyfile @ONLY)
    add_custom_target(docs-lpmp
            COMMAND ${DOXYGEN_EXECUTABLE} Doxyfile
            COMMENT "Generating API documentation with Doxygen"
            VERBATIM)
    if(NOT TARGET docs)
    add_custom_target(docs ALL)
    endif()
    add_dependencies(docs docs-lpmp)
endif()
