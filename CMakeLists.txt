cmake_minimum_required(VERSION 3.22)

project(imgui)

######## ######## ######## ######## ######## ######## ######## ########
# Compiler configuration
######## ######## ######## ######## ######## ######## ######## ########
# Check C++17 support and activate
include(CheckCXXCompilerFlag)
if(UNIX)
  CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
  if(COMPILER_SUPPORTS_CXX17)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
    set(CMAKE_CXX_STANDARD 17) # C++14...
    set(CMAKE_CXX_STANDARD_REQUIRED ON) #...is required...
    set(CMAKE_CXX_EXTENSIONS OFF) #...without compiler extensions like gnu++11
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has C++17 support.")
  else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++17 support. Please use a different C++ compiler.")
  endif()
elseif(MSVC) # Use Visual Studio as a compiler which comes with C++17!
  CHECK_CXX_COMPILER_FLAG("/std:c++17" COMPILER_SUPPORTS_CXX17)
  if (COMPILER_SUPPORTS_CXX17)
    add_compile_options("/std:c++17")
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has C++17 support.")
  else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++17 support. Please use a different C++ compiler.")
  endif()
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /DDEBUG")
endif()
add_compile_definitions("_SILENCE_STDEXT_ARR_ITERS_DEPRECATION_WARNING") # To supress C4996 warning

message(STATUS "CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS_DEBUG: ${CMAKE_CXX_FLAGS_DEBUG}")
message(STATUS "CMAKE_CXX_FLAGS_RELEASE: ${CMAKE_CXX_FLAGS_RELEASE}")

if(MSVC)
  set(WIN32_FLAG WIN32)
endif()
######## ######## ######## ######## ######## ######## ######## ########


######## ######## ######## ######## ######## ######## ######## ########
# Adding executable
######## ######## ######## ######## ######## ######## ######## ########
add_executable(${PROJECT_NAME} ${WIN32_FLAG}
        imgui.cpp
        imgui_demo.cpp
        imgui_draw.cpp
        imgui_widgets.cpp
        imgui_tables.cpp
        examples/example_sdl2_opengl3/main.cpp
        backends/imgui_impl_sdl2.cpp
        backends/imgui_impl_opengl3.cpp
#        examples/libs/gl3w/GL/gl3w.cpp
)
#set_source_files_properties(imgui/examples/libs/gl3w/GL/gl3w.c PROPERTIES COMPILE_FLAGS -Wno-pedantic)
######## ######## ######## ######## ######## ######## ######## ########


######## ######## ######## ######## ######## ######## ######## ########
# Linking libraries
######## ######## ######## ######## ######## ######## ######## ########
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
target_include_directories(${PROJECT_NAME} PRIVATE ${PROJECT_SOURCE_DIR}/include)

## SDL2
find_package(SDL2 CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME}
        PRIVATE
        $<TARGET_NAME_IF_EXISTS:SDL2::SDL2main>
        $<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static>
)

## GLEW
find_package(GLEW REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE GLEW::GLEW)

## GLM
find_package(glm CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE glm::glm)

## GLFW
if (UNIX) # UNIX AND APPLE
  find_package(glfw3 REQUIRED)
  message(STATUS "glfw3: ${GLFW3_FOUND}")
  target_link_libraries(${PROJECT_NAME} PRIVATE glfw)
elseif(WIN32) # vcpkg
  find_package(glfw3 REQUIRED)
endif()
#target_link_libraries(${PROJECT_NAME} PRIVATE glfw)

## ImGui
target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        ${PROJECT_SOURCE_DIR}
        backends
)

######## ######## ######## ######## ######## ######## ######## ########
