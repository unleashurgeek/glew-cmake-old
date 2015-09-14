project(GLEW C)

cmake_minimum_required(VERSION 2.8.9)

if (CMAKE_VERSION VERSION_EQUAL "3.0" OR CMAKE_VERSION VERSION_GREATER "3.0")
    # Until all major package systems have moved to CMake 3,
    # we stick with the older INSTALL_NAME_DIR mechanism
    cmake_policy(SET CMP0042 OLD)
endif()

set(GLEW_VERSION_MAJOR "1")
set(GLEW_VERSION_MINOR "13")
set(GLEW_VERSION_PATCH "0")
set(GLEW_VERSION "${GLEW_VERSION_MAJOR}.${GLEW_VERSION_MINOR}.${GLEW_VERSION_PATCH}")
set(LIB_SUFFIX "" CACHE STRING "Takes an empty string or 64. Directory wehere lib will be installed: lib or lib64")

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(GLEW_BUILD_INFO "Build glewinfo & visualinfo binaries" ON)
option(GLEW_USE_GLU "Build using GLU library" OFF)
option(GLEW_INSTALL "Generate installation target" ON)

if (MSVC)
    option(USE_MSVC_RUNTIME_LIBRARY_DLL "Use MSVC runtime library DLL" ON)
endif()

find_package(OpenGL REQUIRED)
list(APPEND glew_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}/include")

#--------------------------------------------------------------------
# Set compiler specific flags
#--------------------------------------------------------------------
if(WIN32)
	add_definitions(-DWIN32_MEAN_AND_LEAN -DVC_EXTRALEAN -DGLEW_BUILD)
endif()

if(NOT GLEW_USE_GLU)
	add_definitions(-DGLEW_NO_GLU)
endif()

if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    
    if (NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
        foreach (flag CMAKE_C_FLAGS
                        CMAKE_C_FLAGS_DEBUG
                        CMAKE_C_FLAGS_RELEASE
                        CMAKE_C_FLAGS_MINSIZEREL
                        CMAKE_C_FLAGS_RELWITHDEBINFO)

            if (${flag} MATCHES "/MD")
                string(REGEX REPLACE "/MD" "/MT" ${flag} "${${flag}}")
            endif()
            if (${flag} MATCHES "/MDd")
                string(REGEX REPLACE "/MDd" "/MTd" ${flag} "${${flag}}")
            endif()
        endforeach()
    endif()
endif()

if (WIN32)
    list(APPEND glew_LIBRARIES "${OPENGL_gl_LIBRARY}")
    list(APPEND glew_INCLUDE_DIRS "${OPENGL_INCLUDE_DIR}")
elseif (APPLE)
    list(APPEND glew_LIBRARIES "${OPENGL_gl_LIBRARY}")
    list(APPEND glew_LIBRARIES "${AGL_LIBRARY}")
    list(APPEND glew_INCLUDE_DIRS "${OPENGL_INCLUDE_DIR}")
else()
    list(APPEND glew_LIBRARIES "${OPENGL_gl_LIBRARY}")
    list(APPEND glew_LIBRARIES "${X11_LIBRARIES}")
    list(APPEND glew_INCLUDE_DIRS "${OPENGL_INCLUDE_DIR}")
endif()

#--------------------------------------------------------------------
# Choose library output name
#--------------------------------------------------------------------
set(GLEW_LIB_NAME glew)

#--------------------------------------------------------------------
# Create generated files
#--------------------------------------------------------------------
include(CMakePackageConfigHelpers)

if (UNIX)
    set(GLEW_CONFIG_PATH "${CMAKE_INSTALL_PREFIX}/lib/cmake/glew/")
else()
    set(GLEW_CONFIG_PATH "${CMAKE_INSTALL_PREFIX}/")
endif()

configure_package_config_file("${GLEW_SOURCE_DIR}/src/glewConfig.cmake.in"
                              "${GLEW_BINARY_DIR}/src/glewConfig.cmake"
                              INSTALL_DESTINATION "${GLEW_CONFIG_PATH}"
                              PATH_VARS CMAKE_INSTALL_PREFIX
                              NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file("${GLEW_BINARY_DIR}/src/glewConfigVersion.cmake"
                                 VERSION ${GLEW_VERSION}
                                 COMPATIBILITY SameMajorVersion)

configure_file("${GLEW_SOURCE_DIR}/src/glew.pc.in"
               "${GLEW_BINARY_DIR}/src/glew.pc" @ONLY)

#--------------------------------------------------------------------
# Add subdirectories
#--------------------------------------------------------------------
add_subdirectory(src)

#--------------------------------------------------------------------
# Install files other than the library
# The library is installed by src/CMakeLists.txt
#--------------------------------------------------------------------
if (GLEW_INSTALL)
    install(DIRECTORY include/GL DESTINATION include
            FILES_MATCHING PATTERN glew.h PATTERN glxew.h PATTERN wglew.h)

    install(FILES "${GLEW_BINARY_DIR}/src/glewConfig.cmake"
                  "${GLEW_BINARY_DIR}/src/glewConfigVersion.cmake"
            DESTINATION lib${LIB_SUFFIX}/cmake/glew)

    install(EXPORT glewTargets DESTINATION lib${LIB_SUFFIX}/cmake/glew)
    install(FILES "${GLEW_BINARY_DIR}/src/glew.pc}"
            DESTINATION lib${LIB_SUFFIX}/pkgconfig)
endif()