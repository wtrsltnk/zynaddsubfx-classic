#checking include/library paths
message(STATUS "Checking Include Path" $ENV{CMAKE_INCLUDE_PATH} ${CMAKE_INCLUDE_PATH})
message(STATUS "Checking Library Path" $ENV{CMAKE_LIBRARY_PATH} ${CMAKE_LIBRARY_PATH})

#Dependency check
find_package(PkgConfig REQUIRED)
find_package(zlib REQUIRED)
find_package(FFTW REQUIRED)
find_package(Threads   REQUIRED)
find_package(OSS)
find_package(Alsa)
pkg_check_modules(JACK jack)
find_package(PORTAUDIO)
set(FLTK_SKIP_OPENGL true)
pkg_check_modules(NTK ntk)
pkg_check_modules(NTK_IMAGES ntk_images)
find_package(FLTK)
find_package(OpenGL) #for FLTK
find_package(CxxTest)
if(CXXTEST_FOUND)
    set(CXXTEST_USE_PYTHON TRUE)
endif()
# lash
pkg_search_module(LASH lash-1.0)
mark_as_advanced(LASH_LIBRARIES)
pkg_search_module(DSSI dssi>=0.9.0)
mark_as_advanced(DSSI_LIBRARIES)
pkg_search_module(LIBLO liblo>=0.26)
mark_as_advanced(LIBLO_LIBRARIES)

if(NOT WIN32)
    CHECK_FUNCTION_EXISTS(sched_setscheduler HAVE_SCHEDULER)

    execute_process(COMMAND echo fistpl 0
                    COMMAND as -
                    ERROR_VARIABLE AVOID_ASM)
endif()

######### Settings ###########
# NOTE: These cache variables should normally not be changed in this
# file, but either in in CMakeCache.txt before compile, or by passing
# parameters directly into cmake using the -D flag.
set(GuiModule fltk CACHE STRING
    "GUI module, either fltk, ntk or off")
set(CompileTests ${CXXTEST_FOUND} CACHE BOOL
    "whether tests should be compiled in or not")
set(AlsaEnable ${ALSA_FOUND} CACHE BOOL
    "Enable support for Advanced Linux Sound Architecture")
set(JackEnable ${JACK_FOUND} CACHE BOOL
    "Enable support for JACK Audio Connection toolKit")
set(OssEnable ${OSS_FOUND} CACHE BOOL
    "Enable support for Open Sound System")
set(PaEnable ${PORTAUDIO_FOUND} CACHE BOOL
    "Enable support for Port Audio System")
set(LashEnable ${LASH_FOUND} CACHE BOOL
    "Enable LASH Audio Session Handler")
set(DssiEnable ${DSSI_FOUND} CACHE BOOL
    "Enable DSSI Plugin compilation")
set(LibloEnable ${LIBLO_FOUND} CACHE BOOL
    "Enable Liblo")

# Now, handle the incoming settings and set define flags/variables based
# on this

# Add version information
add_definitions(-DVERSION="${VERSION}")

message(STATUS "Building on a '${CMAKE_SYSTEM_NAME}' System")

if(NOT "Darwin" STREQUAL ${CMAKE_SYSTEM_NAME})
    # Add scheduler function existance info (OSX compatiability)
    add_definitions(-DHAVE_SCHEDULER=${HAVE_SCHEDULER})
endif()


# Give a good guess on the best Input/Output default backends
if(JackEnable)
    set(DefaultOutput jack CACHE STRING
        "Default Output module: [null, alsa, oss, jack, portaudio]")
    # Override with perhaps more helpful midi backends
    if(AlsaEnable)
        set(DefaultInput alsa CACHE STRING
            "Default Input module: [null, alsa, oss, jack]")
    elseif(OssEnable)
        set(DefaultInput oss CACHE STRING
            "Default Input module: [null, alsa, oss, jack]")
    else()
        set(DefaultInput jack CACHE STRING
            "Default Input module: [null, alsa, oss, jack]")
    endif()
elseif(AlsaEnable)
    set(DefaultOutput alsa CACHE STRING
        "Default Output module: [null, alsa, oss, jack, portaudio]")
    set(DefaultInput alsa CACHE STRING
        "Default Input module: [null, alsa, oss, jack]")
elseif(OssEnable)
    set(DefaultOutput oss CACHE STRING
        "Default Output module: [null, alsa, oss, jack, portaudio]")
    set(DefaultInput oss CACHE STRING
        "Default Input module: [null, alsa, oss, jack]")
elseif(PaEnable)
    set(DefaultOutput PA CACHE STRING
        "Default Output module: [null, alsa, oss, jack, portaudio]")
    set(DefaultInput RT CACHE STRING
        "Default Input module: [null, alsa, oss, jack]")
else()
    set(DefaultOutput null CACHE STRING
        "Default Output module: [null, alsa, oss, jack, portaudio]")
    set(DefaultInput null CACHE STRING
        "Default Input module: [null, alsa, oss, jack]")
endif()



if(GuiModule STREQUAL qt AND QT_FOUND)
    set(QtGui TRUE)
elseif(GuiModule STREQUAL ntk AND NTK_FOUND)
    set(NtkGui TRUE)
elseif(GuiModule STREQUAL fltk AND FLTK_FOUND)
    set(FltkGui TRUE)
elseif(GuiModule STREQUAL off)
    add_definitions(-DDISABLE_GUI)
else  ()
    set(GuiModule off CACHE STRING "GUI module, either fltk, qt or off")
    add_definitions(-DDISABLE_GUI)
    message(STATUS "GUI module defaulting to off")
endif()

#Build Flags
option (BuildForAMD_X86_64 "Build for AMD x86_64 system" OFF)
option (BuildForCore2_X86_64 "Build for Intel Core2 x86_64 system" OFF)
option (BuildForDebug "Include gdb debugging support" OFF)

set(CMAKE_BUILD_TYPE "Release")

set(BuildOptions_x86_64AMD
    "-O3 -march=athlon64 -m64 -Wall -ffast-math -fno-finite-math-only -fomit-frame-pointer" CACHE STRING
    "X86_64 compiler options")

set(BuildOptions_X86_64Core2
    "-O3 -march=core2 -m64 -Wall -ffast-math -fno-finite-math-only -fomit-frame-pointer" CACHE STRING
    "X86_64 compiler options")

set(BuildOptionsBasic
    "-O3 -msse -msse2 -mfpmath=sse -ffast-math -fomit-frame-pointer" CACHE STRING
    "basic X86 complier options")

set(BuildOptionsDebug
    "-O0 -g3 -ggdb -Wall -Wpointer-arith" CACHE STRING
    "Debug build flags")

########### Settings dependant code ###########
# From here on, the setting variables have  been prepared so concentrate
# on the actual compiling.

if(AlsaEnable)
    list(APPEND AUDIO_LIBRARIES ${ASOUND_LIBRARY})
    list(APPEND AUDIO_LIBRARY_DIRS ${ASOUND_LIBRARY_DIRS})
    add_definitions(-DALSA=1)
endif(AlsaEnable)

if(JackEnable)
    list(APPEND AUDIO_LIBRARIES ${JACK_LIBRARIES})
    list(APPEND AUDIO_LIBRARY_DIRS ${JACK_LIBRARY_DIRS})
    add_definitions(-DJACK=1)
endif(JackEnable)

if(OssEnable)
    add_definitions(-DOSS=1)
endif(OssEnable)

if(PaEnable)
    include_directories(${PORTAUDIO_INCLUDE_DIR})
    add_definitions(-DPORTAUDIO=1)
    list(APPEND AUDIO_LIBRARIES ${PORTAUDIO_LIBRARIES})
    list(APPEND AUDIO_LIBRARY_DIRS ${PORTAUDIO_LIBRARY_DIRS})
endif()

if(CompileTests)
    ENABLE_TESTING()
endif()

if(LashEnable)
    include_directories(${LASH_INCLUDE_DIRS})
    add_definitions(-DLASH=1)
    list(APPEND AUDIO_LIBRARIES ${LASH_LIBRARIES})
    list(APPEND AUDIO_LIBRARY_DIRS ${LASH_LIBRARY_DIRS})
    message(STATUS "Compiling with lash")
endif()

if(LibloEnable)
    include_directories(${LIBLO_INCLUDE_DIRS})
    add_definitions(-DUSE_NSM=1)
    list(APPEND AUDIO_LIBRARIES ${LIBLO_LIBRARIES})
    list(APPEND AUDIO_LIBRARY_DIRS ${LIBLO_LIBRARY_DIRS})
    message(STATUS "Compiling with liblo")
endif()

# other include directories
include_directories(${ZLIB_INCLUDE_DIRS} ${MXML_INCLUDE_DIRS})

if(BuildForDebug)
    set(CMAKE_BUILD_TYPE "Debug")
    set(CMAKE_CXX_FLAGS_DEBUG ${BuildOptionsDebug})
    message (STATUS "Building for ${CMAKE_BUILD_TYPE}, flags: ${CMAKE_CXX_FLAGS_DEBUG}")
else(BuildForDebug)
    set(CMAKE_BUILD_TYPE "Release")
    if(BuildForAMD_X86_64)
            set(CMAKE_CXX_FLAGS_RELEASE ${BuildOptions_x86_64AMD})
    else(BuildForAMD_X86_64)
            if(BuildForCore2_X86_64)
                    set(CMAKE_CXX_FLAGS_RELEASE ${BuildOptions_X86_64Core2})
            else(BuildForCore2_X86_64)
                    set(CMAKE_CXX_FLAGS_RELEASE ${BuildOptionsBasic})
            endif(BuildForCore2_X86_64)
    endif(BuildForAMD_X86_64)
    message (STATUS "Building for ${CMAKE_BUILD_TYPE}, flags: ${CMAKE_CXX_FLAGS_RELEASE}")
endif(BuildForDebug)

if(FLTK_FOUND)
    mark_as_advanced(FORCE FLTK_BASE_LIBRARY)
    mark_as_advanced(FORCE FLTK_CONFIG_SCRIPT)
    mark_as_advanced(FORCE FLTK_DIR)
    mark_as_advanced(FORCE FLTK_FLUID_EXECUTABLE)
    mark_as_advanced(FORCE FLTK_FORMS_LIBRARY)
    mark_as_advanced(FORCE FLTK_GL_LIBRARY)
    mark_as_advanced(FORCE FLTK_IMAGES_LIBRARY)
    mark_as_advanced(FORCE FLTK_INCLUDE_DIR)
    mark_as_advanced(FORCE FLTK_MATH_LIBRARY)
endif(FLTK_FOUND)

if(NTK_FOUND)
    mark_as_advanced(FORCE NTK_BASE_LIBRARY)
    mark_as_advanced(FORCE NTK_CONFIG_SCRIPT)
    mark_as_advanced(FORCE NTK_DIR)
    mark_as_advanced(FORCE FLTK_FLUID_EXECUTABLE)
    mark_as_advanced(FORCE NTK_FORMS_LIBRARY)
    mark_as_advanced(FORCE NTK_GL_LIBRARY)
    mark_as_advanced(FORCE NTK_IMAGES_LIBRARY)
    mark_as_advanced(FORCE NTK_INCLUDE_DIR)
    mark_as_advanced(FORCE NTK_MATH_LIBRARY)
endif(NTK_FOUND)

if(FltkGui)
    if(NOT WIN32)
        #UGLY WORKAROUND
        find_program (FLTK_CONFIG fltk-config)
        if(FLTK_CONFIG)
            execute_process (COMMAND ${FLTK_CONFIG} --use-images --ldflags OUTPUT_VARIABLE FLTK_LDFLAGS)
            string(STRIP "${FLTK_LDFLAGS}" FLTK_LIBRARIES)
        endif()
    endif()

    message(STATUS ${FLTK_LDFLAGS})

    set(GUI_LIBRARIES ${FLTK_LIBRARIES} ${FLTK_LIBRARIES} ${OPENGL_LIBRARIES} zynaddsubfx_gui)

    add_definitions(-DFLTK_GUI)

    message(STATUS "Will build FLTK gui")

    include_directories(
        ${FLTK_INCLUDE_DIR}
        "${CMAKE_CURRENT_SOURCE_DIR}/UI"
        "${CMAKE_CURRENT_BINARY_DIR}/UI")

    add_subdirectory(UI)
endif()

if(NtkGui)

    find_program( FLTK_FLUID_EXECUTABLE ntk-fluid)

    message(STATUS ${NTK_LDFLAGS} ${NTK_IMAGES_LDFLAGS})

    set(GUI_LIBRARIES ${NTK_LIBRARIES} ${NTK_IMAGES_LIBRARIES} ${OPENGL_LIBRARIES} zynaddsubfx_gui)

    add_definitions(-DNTK_GUI)

    message(STATUS "Will build NTK gui")

    include_directories(
        ${NTK_INCLUDE_DIRS}
        "${CMAKE_CURRENT_SOURCE_DIR}/UI"
        "${CMAKE_CURRENT_BINARY_DIR}/UI")

    add_subdirectory(UI)
endif()

########### General section ##############
# Following this should be only general compilation code, and no mention
# of module-specific variables

add_subdirectory(Nio)
add_subdirectory(MXml)

add_library(zynaddsubfx_core STATIC
    globals.h)

target_sources(zynaddsubfx_core
    PRIVATE DSP/AnalogFilter.cpp
    PRIVATE DSP/FFTwrapper.cpp
    PRIVATE DSP/Filter.cpp
    PRIVATE DSP/FormantFilter.cpp
    PRIVATE DSP/SVFilter.cpp
    PRIVATE DSP/Unison.cpp)

target_sources(zynaddsubfx_core
    PRIVATE Effects/Alienwah.cpp
    PRIVATE Effects/Chorus.cpp
    PRIVATE Effects/Distorsion.cpp
    PRIVATE Effects/DynamicFilter.cpp
    PRIVATE Effects/Echo.cpp
    PRIVATE Effects/Effect.cpp
    PRIVATE Effects/EffectLFO.cpp
    PRIVATE Effects/EffectMgr.cpp
    PRIVATE Effects/EQ.cpp
    PRIVATE Effects/Phaser.cpp
    PRIVATE Effects/Reverb.cpp)

target_sources(zynaddsubfx_core
    PRIVATE Misc/Bank.cpp
    PRIVATE Misc/Config.cpp
    PRIVATE Misc/Dump.cpp
    PRIVATE Misc/Master.cpp
    PRIVATE Misc/Microtonal.cpp
    PRIVATE Misc/Part.cpp
    PRIVATE Misc/Util.cpp
    PRIVATE Misc/XMLwrapper.cpp
    PRIVATE Misc/Recorder.cpp
    PRIVATE Misc/WavFile.cpp
    PRIVATE Misc/WaveShapeSmps.cpp)

target_sources(zynaddsubfx_core
    PRIVATE Params/ADnoteParameters.cpp
    PRIVATE Params/Controller.cpp
    PRIVATE Params/EnvelopeParams.cpp
    PRIVATE Params/FilterParams.cpp
    PRIVATE Params/LFOParams.cpp
    PRIVATE Params/PADnoteParameters.cpp
    PRIVATE Params/Presets.cpp
    PRIVATE Params/PresetsArray.cpp
    PRIVATE Params/PresetsStore.cpp
    PRIVATE Params/SUBnoteParameters.cpp)

if(LashEnable)
    target_sources(zynaddsubfx_core
        PRIVATE Misc/LASHClient.cpp)
endif()

target_sources(zynaddsubfx_core
    PRIVATE Synth/SynthNote.cpp
    PRIVATE Synth/ADnote.cpp
    PRIVATE Synth/Envelope.cpp
    PRIVATE Synth/LFO.cpp
    PRIVATE Synth/OscilGen.cpp
    PRIVATE Synth/PADnote.cpp
    PRIVATE Synth/Resonance.cpp
    PRIVATE Synth/SUBnote.cpp)

target_include_directories(zynaddsubfx_core
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
    PUBLIC ${CMAKE_CURRENT_BINARY_DIR}
    PUBLIC MXml)

target_link_directories(zynaddsubfx_core
    PRIVATE ${AUDIO_LIBRARY_DIRS}
    PRIVATE ${ZLIB_LIBRARY_DIRS}
    PRIVATE ${FFTW_LIBRARY_DIRS}
    PRIVATE ${FLTK_LIBRARY_DIRS}
    PRIVATE ${NTK_LIBRARY_DIRS})

target_link_libraries(zynaddsubfx_core
    PRIVATE ${ZLIB_LIBRARIES}
    PRIVATE ${FFTW_LIBRARIES}
    PRIVATE ${MXML_LIBRARIES}
    PRIVATE ${OS_LIBRARIES}
    PRIVATE MXml
    PRIVATE pthread)

if(NOT AVOID_ASM)
    message(STATUS "Compiling with x86 opcode support")
    target_compile_definitions(zynaddsubfx_core
        PUBLIC -DASM_F2I_YES)
endif()

add_executable(zynaddsubfx
    main.cpp)

target_link_libraries(zynaddsubfx
    PRIVATE zynaddsubfx_nio
    PRIVATE zynaddsubfx_core
    PRIVATE ${GUI_LIBRARIES}
    PRIVATE ${NIO_LIBRARIES}
    PRIVATE ${AUDIO_LIBRARIES})

if(WIN32)
    find_file(FFTW_BIN_PATH
        NAME "libfftw3-3.dll"
        PATHS ${CMAKE_PREFIX_PATH}/bin)

    configure_file(${FFTW_BIN_PATH} ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)

    if(PaEnable)
        find_file(PORTAUDIO_BIN_PATH
            NAME "libportaudio.dll"
            PATHS ${CMAKE_PREFIX_PATH}/bin)

        configure_file(${PORTAUDIO_BIN_PATH} ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
    endif(PaEnable)
endif(WIN32)

if(DssiEnable)
    add_library(zynaddsubfx_dssi SHARED
        Output/DSSIaudiooutput.cpp)

    target_link_libraries(zynaddsubfx_dssi
        PRIVATE zynaddsubfx_core
        PRIVATE ${OS_LIBRARIES})

    if(${CMAKE_SIZEOF_VOID_P} EQUAL "8")
        install(
            TARGETS zynaddsubfx_dssi
            LIBRARY DESTINATION lib64/dssi/)
    else()
        install(
            TARGETS zynaddsubfx_dssi
            LIBRARY DESTINATION lib/dssi/)
    endif()
endif()

message(STATUS "Link libraries: ${ZLIB_LIBRARY} ${FFTW_LIBRARY} ${MXML_LIBRARIES} ${AUDIO_LIBRARIES} ${OS_LIBRARIES}")

install(
    TARGETS zynaddsubfx
    RUNTIME DESTINATION bin)

if(NtkGui)
    install(
        DIRECTORY ../pixmaps
        DESTINATION share/zynaddsubfx)

    target_compile_definitions(zynaddsubfx
        PRIVATE -DPIXMAP_PATH="${CMAKE_INSTALL_PREFIX}/share/zynaddsubfx/pixmaps/")

    target_compile_definitions(zynaddsubfx
        PRIVATE -DSOURCE_DIR="${CMAKE_CURRENT_SOURCE_DIR}")
endif(NtkGui)

include(CTest)
