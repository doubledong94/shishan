cmake_minimum_required(VERSION 3.16)
project(shishan)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Find required packages
find_package(OpenGL REQUIRED)

# Try to find GLFW first
find_package(glfw3 QUIET)

# Platform-specific settings
if(APPLE)
    find_library(COCOA_LIBRARY Cocoa)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(COREVIDEO_LIBRARY CoreVideo)
    set(EXTRA_LIBS ${COCOA_LIBRARY} ${IOKIT_LIBRARY} ${COREVIDEO_LIBRARY})
elseif(UNIX) # Linux
    find_package(X11 REQUIRED)
    set(EXTRA_LIBS X11::X11 X11::Xxf86vm X11::Xrandr X11::Xi)
endif()

# Set dependencies directory outside of build
set(DEPS_DIR "${CMAKE_SOURCE_DIR}/deps")
set(FETCHCONTENT_BASE_DIR "${DEPS_DIR}")

# Download and build GLFW if not found
if(NOT glfw3_FOUND)
    include(FetchContent)
    FetchContent_Declare(
        glfw
        GIT_REPOSITORY https://github.com/glfw/glfw.git
        GIT_TAG 3.3.8  # Specific version for reproducibility
        SOURCE_DIR "${DEPS_DIR}/glfw-src"
        BINARY_DIR "${DEPS_DIR}/glfw-build"
    )
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(glfw)
    set(GLFW_LIB glfw)
endif()

# Download and build imgui
include(FetchContent)
FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG v1.89.9  # Specific version for reproducibility
    SOURCE_DIR "${DEPS_DIR}/imgui-src"
    BINARY_DIR "${DEPS_DIR}/imgui-build"
)
FetchContent_MakeAvailable(imgui)

# Create imgui library
set(IMGUI_SOURCES
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/imgui_demo.cpp
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp
)

add_library(imgui STATIC ${IMGUI_SOURCES})
target_include_directories(imgui PUBLIC
    ${imgui_SOURCE_DIR}
    ${imgui_SOURCE_DIR}/backends
)
target_link_libraries(imgui PUBLIC ${GLFW_LIB} OpenGL::GL)

# Create main executable
add_executable(shishan src/main.cpp)
target_link_libraries(shishan imgui ${GLFW_LIB} OpenGL::GL ${EXTRA_LIBS})

# Create src directory if it doesn't exist
file(MAKE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src)