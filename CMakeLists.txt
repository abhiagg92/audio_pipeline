cmake_minimum_required(VERSION 3.16)
set(CMAKE_VERBOSE_MAKEFILE True)

project(audio_pipeline VERSION 1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FindPkgConfig)

set(CMAKE_CXX_FLAGS_DEBUG "-ggdb -Wall -fPIC -Wno-overloaded-virtual -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}")
set(CMAKE_CXX_FLAGE_RELEASE "-O3 -DNDEBUG -Wall -fPIC -Wno-overloaded-virtual -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-ggdb -O3 -Wall -fPIC -Wno-overloaded-virtual -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}")
set(ILLIXR_INTEGRATION ON)
set(ILLIXR_ROOT "" CACHE PATH "Path to ILLIXR headers")

pkg_check_modules(PORTAUDIO portaudio-2.0>=19)
pkg_check_modules(SPATIALAUDIO spatialaudio)

if(DEFINED INTERNAL_OPENCV)
    message("USING INTERNAL ${INTERNAL_OPENCV}")
    find_package(OpenCV REQUIRED PATHS ${INTERNAL_OPENCV} NO_DEFAULT_PATH NO_CMAKE_ENVIRONMENT_PATH NO_SYSTEM_ENVIRONMENT_PATH NO_CMAKE_PACKAGE_REGISTRY NO_CMAKE_SYSTEM_PATH NO_CMAKE_SYSTEM_PACKAGE_REGISTRY)
else()
    find_package(OpenCV REQUIRED)
endif()
add_executable(solo${ILLIXR_BUILD_SUFFIX}.exe src/main.cpp)
add_library(plugin.audio_pipeline${ILLIXR_BUILD_SUFFIX} SHARED src/audio.cpp src/audio_component.cpp src/sound.cpp src/realtime.cpp)
include_directories(${CMAKE_INSTALL_PREFIX}/include ${PROJECT_SOURCE_DIR}/include ${ILLIXR_ROOT} ${OpenCV_INCLUDE_DIRS})
if(ILLIXR_INTEGRATION)
    target_compile_definitions(plugin.audio_pipeline${ILLIXR_BUILD_SUFFIX} PUBLIC -DAUDIO_SAMPLES=\"${CMAKE_INSTALL_PREFIX}/etc/audio_pipeline\")
endif()
target_include_directories(plugin.audio_pipeline${ILLIXR_BUILD_SUFFIX} PUBLIC ${PORTAUDIO_INCLUDE_DIRS} ${SPATIALAUDIO_INCLUDE_DIRS})
target_link_libraries(plugin.audio_pipeline${ILLIXR_BUILD_SUFFIX} PUBLIC ${PORTAUDIO_LIBRARIES} ${SPATIALAUDIO_LIBRARIES} ${OpenCV_LIBRARIES})

target_link_libraries(solo${ILLIXR_BUILD_SUFFIX}.exe PUBLIC plugin.audio_pipeline${ILLIXR_BUILD_SUFFIX} pthread)

install(TARGETS plugin.audio_pipeline${ILLIXR_BUILD_SUFFIX} DESTINATION lib)
install(TARGETS solo${ILLIXR_BUILD_SUFFIX}.exe DESTINATION bin)
install(DIRECTORY ${CMAKE_SOURCE_DIR}/samples
        DESTINATION etc/audio_pipeline)
