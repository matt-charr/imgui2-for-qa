#################################################################################
# Copyright (c) 2021, Savely Pototsky (SavaLione)
# All rights reserved.
#################################################################################
# Checking the minimum version
cmake_minimum_required (VERSION 3.13.1)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Project name
set(IMGUI_PROJECT "imgui")
set(IMGUI_PROJECT_VERSION "1.84.2.8275456")

project(${IMGUI_PROJECT}
    VERSION ${IMGUI_PROJECT_VERSION}
    LANGUAGES CXX
)

#################################################################################
#                                     Options                                   #
#################################################################################
option(IMGUI_BUILD_STATIC "Build static library" OFF)

option(IMGUI_BUILD_BACKEND_ALLEGRO5 "Build the allegro5 backend" OFF)
option(IMGUI_BUILD_BACKEND_ANDROID "Build the android backend" OFF)
option(IMGUI_BUILD_BACKEND_DX9 "Build the dx9 backend" OFF)
option(IMGUI_BUILD_BACKEND_DX10 "Build the dx10 backend" OFF)
option(IMGUI_BUILD_BACKEND_DX11 "Build the dx11 backend" OFF)
option(IMGUI_BUILD_BACKEND_DX12 "Build the dx12 backend" OFF)
option(IMGUI_BUILD_BACKEND_GLFW "Build the glfw backend" ON)
option(IMGUI_BUILD_BACKEND_GLUT "Build the glut backend" OFF)
option(IMGUI_BUILD_BACKEND_MARMALADE "Build the marmalade backend" OFF)
option(IMGUI_BUILD_BACKEND_METAL "Build the metal backend" OFF)
option(IMGUI_BUILD_BACKEND_OPENGL2 "Build the opengl2 backend" OFF)
option(IMGUI_BUILD_BACKEND_OPENGL3 "Build the opengl2 backend" OFF)
option(IMGUI_BUILD_BACKEND_OPENGL3_LOADER "Build the opengl3 loader backend" OFF)
option(IMGUI_BUILD_BACKEND_OSX "Build the osx backend" OFF)
option(IMGUI_BUILD_BACKEND_SDL "Build the sdl backend" OFF)
option(IMGUI_BUILD_BACKEND_VULKAN "Build the vulkan backend" OFF)
option(IMGUI_BUILD_BACKEND_WGPU "Build the wgpu backend" OFF)
option(IMGUI_BUILD_BACKEND_WIN32 "Build the win32 backend" OFF)

#################################################################################
#                                      Imgui                                    #
#################################################################################

# source files.
    # imgui.
file(
    GLOB IMGUI_SRC
    ${PROJECT_SOURCE_DIR}/imgui-for-qa/*.cpp
)
    # implot.
file(
    GLOB IMPLOT_SRC
    ${PROJECT_SOURCE_DIR}/implot-for-qa/*.cpp
)
    # imgui-filebrowser.
file(
    GLOB IMFILEBROWSER_SRC
    ${PROJECT_SOURCE_DIR}/imgui-filebrowser-for-qa/*.cpp
)
    # implot3d.
file(
    GLOB IMPLOT3D_SRC
    ${PROJECT_SOURCE_DIR}/implot3d-for-qa/*.cpp
)

# header files.
    # imgui.
file(
    GLOB IMGUI_INC
    ${PROJECT_SOURCE_DIR}/imgui-for-qa/*.h
    ${PROJECT_SOURCE_DIR}/imgui-for-qa/*.hpp
)
    # implot.
file(
    GLOB IMPLOT_INC
    ${PROJECT_SOURCE_DIR}/implot-for-qa/*.h
    ${PROJECT_SOURCE_DIR}/implot-for-qa/*.hpp
)
    # imgui-filebrowser.
file(
    GLOB IMFILEBROWSER_INC
    ${PROJECT_SOURCE_DIR}/imgui-filebrowser-for-qa/*.h
    ${PROJECT_SOURCE_DIR}/imgui-filebrowser-for-qa/*.hpp
)
    # implot3d.
file(
    GLOB IMPLOT3D_INC
    ${PROJECT_SOURCE_DIR}/implot3d-for-qa/*.h
    ${PROJECT_SOURCE_DIR}/implot3d-for-qa/*.hpp
)

if(IMGUI_BUILD_STATIC)
    add_library(${IMGUI_PROJECT} STATIC ${IMGUI_SRC} ${IMPLOT_SRC} ${IMFILEBROWSER_SRC} ${IMPLOT3D_SRC})
else()
    add_library(${IMGUI_PROJECT} SHARED ${IMGUI_SRC} ${IMPLOT_SRC} ${IMFILEBROWSER_SRC} ${IMPLOT3D_SRC})
endif()
target_include_directories(${IMGUI_PROJECT} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui-for-qa>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/implot-for-qa>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui-filebrowser-for-qa>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/implot3d-for-qa>
    $<INSTALL_INTERFACE:include>
    $<INSTALL_INTERFACE:include/implot>
    $<INSTALL_INTERFACE:include/implot3d>
    $<INSTALL_INTERFACE:include/imgui-filebrowser>
)

# install exports

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}_Exports
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        NAMELINK_SKIP
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(
    EXPORT ${PROJECT_NAME}_Exports
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    NAMESPACE ${PROJECT_NAME}::
)
install(TARGETS ${PROJECT_NAME}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        NAMELINK_ONLY
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)
write_basic_package_version_file(
    "${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${IMGUI_PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)
install(
FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
DESTINATION
    ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

#################################################################################
#                                     Backends                                  #
#################################################################################

# OpenGL3
if (IMGUI_BUILD_BACKEND_OPENGL3)
    # imgui OpenGL3
    project(imgui_opengl3)
    set(IMGUI_BACKEND_OPENGL3_SRC imgui-for-qa/backends/imgui_impl_opengl3.cpp)
    # OpenGL
    if (NOT TARGET glfw)
        find_package(OpenGL REQUIRED)
    endif()
    # imgui opengl3
    if(IMGUI_BUILD_STATIC)
	    add_library(imgui_opengl3 STATIC ${IMGUI_BACKEND_OPENGL3_SRC})
    else()
	    add_library(imgui_opengl3 SHARED ${IMGUI_BACKEND_OPENGL3_SRC})
    endif()
    target_link_libraries(imgui_opengl3 imgui)
    target_include_directories(imgui_opengl3 PUBLIC 
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/backends>
        $<INSTALL_INTERFACE:include/backends>
    )
    # OpenGL
    target_include_directories(imgui_opengl3 PUBLIC ${OPENGL_INCLUDE_DIR})
    target_link_libraries(imgui_opengl3 ${OPENGL_gl_LIBRARY})
    # install exports
    install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}_Exports
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            NAMELINK_SKIP
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
    install(
        EXPORT ${PROJECT_NAME}_Exports
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
        NAMESPACE ${PROJECT_NAME}::
    )
    install(TARGETS ${PROJECT_NAME}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            NAMELINK_ONLY
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
    configure_package_config_file(
        "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )
    write_basic_package_version_file(
        "${PROJECT_NAME}ConfigVersion.cmake"
        VERSION ${IMGUI_PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
    )
    install(
        FILES
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION
            ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )
endif()

# GLFW
if (IMGUI_BUILD_BACKEND_GLFW)
    # imgui glfw
    project(imgui_glfw)
    set(IMGUI_BACKEND_GLFW_SRC imgui-for-qa/backends/imgui_impl_glfw.cpp)
    # glfw
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    set(IMGUI_DEPS_GLFW_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/glfw-for-qa/include" CACHE PATH "imgui GLFW lib include dependence")
    if (NOT TARGET glfw AND NOT CW_WEB)
        add_subdirectory(${PROJECT_SOURCE_DIR}/glfw-for-qa)
    endif()
    # imgui glfw
    if(IMGUI_BUILD_STATIC)
	    add_library(imgui_glfw STATIC ${IMGUI_BACKEND_GLFW_SRC})
    else()
	    add_library(imgui_glfw SHARED ${IMGUI_BACKEND_GLFW_SRC})
    endif()
    target_link_libraries(imgui_glfw PUBLIC imgui)
    if (NOT CW_WEB)
        target_link_libraries(imgui_glfw PUBLIC glfw)
    endif ()
    target_include_directories(imgui_glfw PUBLIC 
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/backends>
        $<INSTALL_INTERFACE:include/backends>
    )
    target_include_directories(imgui_glfw PUBLIC ${GLFW_INCLUDE_DIR})
    # install exports
    install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}_Exports
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            NAMELINK_SKIP
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
    install(
        EXPORT ${PROJECT_NAME}_Exports
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
        NAMESPACE ${PROJECT_NAME}::
    )
    install(TARGETS ${PROJECT_NAME}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            NAMELINK_ONLY
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
    configure_package_config_file(
        "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )
    write_basic_package_version_file(
        "${PROJECT_NAME}ConfigVersion.cmake"
        VERSION ${IMGUI_PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
    )
    install(
        FILES
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION
            ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    )
endif()

# install headers.
install(
    FILES ${IMGUI_INC}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(
    FILES ${PROJECT_SOURCE_DIR}/imgui-for-qa/backends/imgui_impl_glfw.h ${PROJECT_SOURCE_DIR}/imgui-for-qa/backends/imgui_impl_opengl3.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/backends
)
install(
    FILES ${IMPLOT_INC}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/implot
)
install(
    FILES ${IMFILEBROWSER_INC}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imgui-filebrowser
)
install(
    FILES ${IMPLOT3D_INC}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/implot3d
)