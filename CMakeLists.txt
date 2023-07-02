cmake_minimum_required(VERSION 3.5)

project(Blackhole)

set(CMAKE_VERBOSE_MAKEFILE ON)


list(APPEND CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})

if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
  message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
  file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/release/0.18/conan.cmake"
                "${CMAKE_BINARY_DIR}/conan.cmake"
                TLS_VERIFY ON)
endif()

include(${CMAKE_BINARY_DIR}/conan.cmake)

conan_cmake_configure(REQUIRES imgui/1.86 glfw/3.3.6 glew/2.2.0 glm/0.9.9.8 stb/cci.20210713
                      GENERATORS cmake_find_package)

conan_cmake_autodetect(settings)

# conan_cmake_install(PATH_OR_REFERENCE .
#                     BUILD missing
#                     REMOTE conancenter
#                     SETTINGS ${settings})

find_package(PkgConfig REQUIRED)
pkg_check_modules(imgui REQUIRED IMPORTED_TARGET imgui)
pkg_check_modules(stb REQUIRED IMPORTED_TARGET stb)
find_package(glfw3 REQUIRED)
find_package(GLEW REQUIRED)
find_package(glm REQUIRED)
find_package(OpenGL REQUIRED)


add_link_options("-lGL") 

file(GLOB SRC_FILES
  "${PROJECT_SOURCE_DIR}/src/*.h"
  "${PROJECT_SOURCE_DIR}/src/*.cpp"
  "${PROJECT_SOURCE_DIR}/src/*.c"
  "${PROJECT_SOURCE_DIR}/src/*.cc")

add_executable(${CMAKE_PROJECT_NAME} ${SRC_FILES})

target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC PkgConfig::imgui)
target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC PkgConfig::stb)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE glfw)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE GLEW::GLEW)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE glm::glm)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OpenGL::GL)

target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_17)

# Copy assets files after build.
add_custom_command(
  TARGET ${CMAKE_PROJECT_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/assets"
          "$<TARGET_FILE_DIR:${CMAKE_PROJECT_NAME}>/assets")

add_custom_command(
  TARGET ${CMAKE_PROJECT_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/shader/"
          "$<TARGET_FILE_DIR:${CMAKE_PROJECT_NAME}>/shader")
