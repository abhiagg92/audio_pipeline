cmake_minimum_required(VERSION 3.16)
set(CMAKE_VERBOSE_MAKEFILE True)
set(CMAKE_BUILD_TYPE RelWithDebInfo)

project(audio_pipeline VERSION 1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(ILLIXR_INTEGRATION "" ON)
include(FindPkgConfig)
set(CMAKE_CXX_FLAGS_DEBUG "-ggdb -Wall -fPIC -Wno-overloaded-virtual -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}")
set(CMAKE_CXX_FLAGE_RELEASE "-O3 -DNDEBUG -Wall -fPIC -Wno-overloaded-virtual -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-ggdb -O3 -Wall -fPIC -Wno-overloaded-virtual -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}")

add_executable(solo.opt.exe src/main.cpp)
add_library(plugin.audio_pipeline SHARED src/audio.cpp src/audio_component.cpp src/sound.cpp src/realtime.cpp)
include_directories(${CMAKE_INSTALL_PREFIX}/include "${PROJECT_SOURCE_DIR}/include")
include(ExternalProject)
pkg_check_modules(PORTAUDIO portaudio>19)

if (NOT PORTAUDIO_FOUND)
    EXTERNALPROJECT_ADD(PORTAUDIO
            GIT_REPOSITORY https://github.com/PortAudio/portaudio.git
            GIT_TAG 7e2a33c875c6b2b53a8925959496cc698765621f
            CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            )
    target_link_libraries(plugin.audio_pipeline portaudio)
else()
    target_include_directories(plugin.audio_pipeline ${PORTAUDIO_INCLUDE_DIR})
    target_link_libraries(plugin.audio_pipeline ${PORTAUDIO_LIBRARIES})
endif()

EXTERNALPROJECT_ADD(SPATIALAUDIO
        GIT_REPOSITORY https://github.com/ILLIXR/libspatialaudio.git
        GIT_TAG 77a901e337a63aa981745ab369ccf597834a37a5
        CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DILLIXR_INTEGRATION=${ILLIXR_INTEGRATION}
        )

target_link_libraries(plugin.audio_pipeline spatialaudio)
target_link_libraries(solo.opt.exe plugin.audio_pipeline pthread)