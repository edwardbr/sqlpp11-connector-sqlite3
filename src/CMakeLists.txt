set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)

add_library(${PROJECT_NAME}
		connection.cpp
		bind_result.cpp
		prepared_statement.cpp
		detail/connection_handle.cpp
      )

if (SQLPP_DYNAMIC_LOADING)
    add_library(${PROJECT_NAME}-dynamic
                    connection.cpp
                    bind_result.cpp
                    prepared_statement.cpp
                    detail/connection_handle.cpp
                    detail/dynamic_libsqlite3.cpp
        )

    get_filename_component(SQLITE3_LIB_FILE ${SQLITE3_LIBRARY} NAME)

    if (WIN32)
        # the import library (.lib) needs to be replaced by .dll
        get_filename_component(SQLITE3_LIB_FILE ${SQLITE3_LIB_FILE} NAME_WE)
        string(APPEND SQLITE3_LIB_FILE ".dll")
    endif()

    set_target_properties(${PROJECT_NAME} PROPERTIES DEFINE_SYMBOL SQLPP11_COMPILING_DLL)
    set_target_properties(${PROJECT_NAME}-dynamic PROPERTIES DEFINE_SYMBOL SQLPP11_COMPILING_DLL)

    target_compile_definitions(${PROJECT_NAME}-dynamic
                               PUBLIC SQLPP_DYNAMIC_LOADING
                               PRIVATE SQLPP_DYNAMIC_LOADING_FILENAME=${SQLITE3_LIB_FILE})

    target_include_directories(${PROJECT_NAME}-dynamic
                               PUBLIC
                               ${SQLPP11_INCLUDE_DIR}
                               ${HinnantDate_INCLUDE_DIR}
															 ${SQLITE3_INCLUDE_DIRS}
                               $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                               $<INSTALL_INTERFACE:include>)
endif()

target_include_directories(${PROJECT_NAME}
                           PUBLIC
                           ${SQLPP11_INCLUDE_DIR}
                           ${date_INCLUDE_DIR}
						   ${SQLITE3_INCLUDE_DIRS}
                           $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                           $<INSTALL_INTERFACE:include>)

if (BUILD_SHARED_LIBS)
    target_compile_definitions(${PROJECT_NAME} PUBLIC SQLPP11_SHARED)
    if (SQLPP_DYNAMIC_LOADING)
        target_compile_definitions(${PROJECT_NAME}-dynamic PUBLIC SQLPP11_SHARED)
    endif()
endif()

if (SQLCIPHER)
    target_compile_definitions(${PROJECT_NAME} PUBLIC SQLPP_USE_SQLCIPHER)
    target_link_libraries(${PROJECT_NAME} sqlcipher::sqlcipher)
    if (SQLPP_DYNAMIC_LOADING)
        target_compile_definitions(${PROJECT_NAME}-dynamic PUBLIC SQLPP_USE_SQLCIPHER)
        target_include_directories(${PROJECT_NAME}-dynamic PUBLIC ${SQLCIPHER_INCLUDE_DIRS})
    endif()
else()
    target_link_libraries(${PROJECT_NAME} ${SQLITE3_LIBRARIES})
endif()

install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)

if (SQLPP_DYNAMIC_LOADING)
    install(TARGETS ${PROJECT_NAME}-dynamic
            ARCHIVE DESTINATION lib
            LIBRARY DESTINATION lib
            RUNTIME DESTINATION bin)
endif()
