# Copyright (c) 2013 - 2016, Roland Bock
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification,
# are permitted provided that the following conditions are met:
#
#   Redistributions of source code must retain the above copyright notice, this
#   list of conditions and the following disclaimer.
#
#   Redistributions in binary form must reproduce the above copyright notice, this
#   list of conditions and the following disclaimer in the documentation and/or
#   other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
# ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
# ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 3.13)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

include(HunterGate)
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.165.tar.gz"
    SHA1 "5a73f91df5f6109c0bb1104d0c0ee423f7bece79"
    LOCAL
)

project (sqlpp11-sqlite3 VERSION 0.29 LANGUAGES CXX)

enable_testing()

set(CMAKE_CXX_STANDARD 11)

option(ENABLE_TESTS "Build unit tests" Off)
option(SQLCIPHER "Build with sqlcipher" Off)

hunter_add_package(date)
find_package(date REQUIRED)

hunter_add_package(Sqlpp11)
find_package(Sqlpp11 REQUIRED)

if(SQLCIPHER )
    message(STATUS "using crypt")

	hunter_add_package(OpenSSL)
	find_package(OpenSSL REQUIRED)

	hunter_add_package(sqlcipher)
	find_package(sqlcipher REQUIRED)

	set (SQLITE3_LIBRARIES "${SQLCIPHER_LIBRARIES}" )
	set (SQLITE3_INCLUDE_DIRS "${SQLCIPHER_INCLUDE_DIRS}" )
    set (SQLITE3_LIBRARY "${SQLCIPHER_LIBRARY}")
    set_property(TARGET sqlcipher::sqlcipher PROPERTY INTERFACE_COMPILE_DEFINITIONS SQLITE_HAS_CODEC)
    set_property(TARGET sqlcipher::sqlcipher APPEND PROPERTY INTERFACE_COMPILE_DEFINITIONS "SQLITE_TEMPSTORE=2")
else()
    message(STATUS "not using crypt")
    hunter_add_package(sqlite3)
	find_package(sqlite3 CONFIG REQUIRED)
endif()

add_library(${PROJECT_NAME} STATIC
		src/connection.cpp
		src/bind_result.cpp

		src/prepared_statement.cpp
		src/detail/connection_handle.cpp
      )

target_link_libraries(${PROJECT_NAME}  
    INTERFACE 
        date_interface 
	PUBLIC
		sqlpp11
) 

target_include_directories(${PROJECT_NAME}  PUBLIC 
							$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
							$<INSTALL_INTERFACE:include> )

if (SQLCIPHER)
    target_compile_definitions(${PROJECT_NAME} PUBLIC SQLPP_USE_SQLCIPHER)
    target_link_libraries(${PROJECT_NAME} PUBLIC sqlcipher::sqlcipher)
else()
    target_link_libraries(${PROJECT_NAME} PUBLIC sqlite3::sqlite3)
endif()

install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)


if(ENABLE_TESTS)
	add_subdirectory(tests)
endif()


#set(pc_config "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc")
#configure_file(${PROJECT_NAME}.pc.in ${pc_config} @ONLY)

#set(la_config "${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}.la")
#configure_file(lib${PROJECT_NAME}.la.in ${la_config} @ONLY)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

set(include_dir "${PROJECT_SOURCE_DIR}/include")
file(GLOB_RECURSE sqlpp_headers ${include_dir}/*.h)

include(CMakePackageConfigHelpers)

# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS ${PROJECT_NAME}
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    FILES ${sqlpp_headers}
    DESTINATION "${include_install_dir}/${PROJECT_NAME}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

#install(FILES ${pc_config} DESTINATION lib/pkgconfig)
#install(FILES ${la_config} DESTINATION lib)

