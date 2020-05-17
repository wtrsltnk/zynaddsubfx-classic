#Defaults:
#         - Wave Output (enabled with the record function)
#         - Null Output
#         - Null Output Running by default
#         - Managed with OutMgr
add_library(zynaddsubfx_nio STATIC
    WavEngine.cpp
    NulEngine.cpp
    AudioOut.cpp
    MidiIn.cpp
    OutMgr.cpp
    InMgr.cpp
    Engine.cpp
    EngineMgr.cpp
    Nio.cpp)

target_include_directories(zynaddsubfx_nio
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
    PUBLIC ${CMAKE_CURRENT_BINARY_DIR}
    PUBLIC ../MXml)

target_compile_definitions(zynaddsubfx_nio
    PRIVATE -DOUT_DEFAULT="${DefaultOutput}"
    PRIVATE -DIN_DEFAULT="${DefaultInput}")

if(WIN32)
    target_sources(zynaddsubfx_nio
        PRIVATE RtEngine.cpp
        PRIVATE RtEngine.h
        PRIVATE RtError.h
        PRIVATE RtMidi.cpp
        PRIVATE RtMidi.h)

    target_compile_definitions(zynaddsubfx_nio
        PRIVATE -D__WINDOWS_MM__)

    target_link_libraries(zynaddsubfx_nio
        PRIVATE winmm)
endif()

if(JackEnable)
    target_sources(zynaddsubfx_nio
        PRIVATE JackEngine.cpp)

    target_include_directories(zynaddsubfx_nio
        PRIVATE ${JACK_INCLUDE_DIR})

    target_link_libraries(zynaddsubfx_nio
        PRIVATE ${JACK_LIBRARIES})
endif(JackEnable)

if(PaEnable)
    target_sources(zynaddsubfx_nio
        PRIVATE PaEngine.cpp)

    target_include_directories(zynaddsubfx_nio
        PRIVATE ${PORTAUDIO_INCLUDE_DIR})

    target_link_libraries(zynaddsubfx_nio
        PRIVATE ${PORTAUDIO_LIBRARIES})
endif(PaEnable)

if(AlsaEnable)
    target_sources(zynaddsubfx_nio
        PRIVATE AlsaEngine.cpp)

    target_link_libraries(zynaddsubfx_nio
        PRIVATE ${ASOUND_LIBRARY})
endif(AlsaEnable)

if(OssEnable)
    target_sources(zynaddsubfx_nio
        PRIVATE OssEngine.cpp)
endif(OssEnable)

