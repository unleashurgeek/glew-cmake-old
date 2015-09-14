
include_directories("${GLEW_SOURCE_DIR}/src"
                    "${GLEW_BINARY_DIR}/src"
                    ${glew_INCLUDE_DIRS})
                    
set(glew_HEADERS "${GLEW_SOURCE_DIR}/include/GL/glew.h"
                 "${GLEW_SOURCE_DIR}/include/GL/glxew.h"
                 "${GLEW_SOURCE_DIR}/include/GL/wglew.h")
                 
set(glew_SOURCES glew.c)

if(MSVC)
    list(APPEND glew_SOURCES "${GLEW_SOURCE_DIR}/build/glew.rc")
endif()

add_library(glew ${glew_SOURCES} ${glew_HEADERS})
set_target_properties(glew PROPERTIES
                      OUTPUT_NAME "${GLEW_LIB_NAME}"
                      VERSION ${GLEW_VERSION}
                      SOVERSION ${GLEW_VERSION_MAJOR}
                      POSITION_INDPENDENT_CODE ON
                      FOLDER "GLEW")
target_link_libraries(glew ${glew_LIBRARIES})

if (GLEW_BUILD_INFO)
    set(glewinfo_SOURCE glewinfo.c)
    set(visualinfo_SOURCE visualinfo.c)
    
    if(MSVC)
        list(APPEND glewinfo_SOURCE "${GLEW_SOURCE_DIR}/build/glewinfo.rc")
        list(APPEND visualinfo_SOURCE "${GLEW_SOURCE_DIR}/build/visualinfo.rc")
    endif()
    
    add_executable(glewinfo ${glewinfo_SOURCE} ${glew_HEADERS})
    add_executable(visualinfo ${visualinfo_SOURCE} ${glew_HEADERS})
    
    target_link_libraries(glewinfo glew)
    target_link_libraries(visualinfo glew)
    set_target_properties(glewinfo visualinfo PROPERTIES
                          FOLDER "GLEW")
endif()

if (BUILD_SHARED_LIBS)
    if (WIN32)
        set_target_properties(glew PROPERTIES PREFIX "" IMPORT_PREFIX "")
        
        set_target_properties(glew PROPERTIES RUNTIME_OUTPUT_NAME "glew")
        
        if (MINGW)
            set_target_properties(glew PROPERTIES IMPORT_SUFFIX "dll.a")
        else()
            set_target_properties(glew PROPERTIES IMPORT_SUFFIX "dll.lib")
        endif()
    elseif (APPLE)
        get_target_property(glew_CFLAGS glew COMPILE_FLAGS)
        if (NOT glew_CFLAGS)
            set(glew_CFLAGS "")
        endif()
        set_target_properties(glew PROPERTIES COMPILE_FLAGS "${glew_CFLAGS} -fno-common"
                              INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}")

        set_target_properties(glew PROPERTIES LIBRARY_OUTPUT_NAME "glew")
    else() 
        set_target_properties(glew PROPERTIES LIBRARY_OUTPUT_NAME "glew")
    endif()
endif()

if (GLEW_INSTALL)
    install(TARGETS glew EXPORT glewTargets DESTINATION lib${LIB_SUFFIX})
    if (GLEW_BUILD_INFO)
        install(TARGETS glewinfo visualinfo DESTINATION bin)
    endif()
endif()