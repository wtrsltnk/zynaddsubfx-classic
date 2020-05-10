set(UI_fl_files
    ADnoteUI.fl
    BankUI.fl
    ConfigUI.fl
    EffUI.fl
    EnvelopeUI.fl
    FilterUI.fl
    LFOUI.fl
    MasterUI.fl
    MicrotonalUI.fl
    OscilGenUI.fl
    PADnoteUI.fl
    PartUI.fl
    PresetsUI.fl
    ResonanceUI.fl
    SUBnoteUI.fl
    VirKeyboard.fl)

set_source_files_properties(UI/MasterUI.h PROPERTIES GENERATED 1)
fltk_wrap_ui(zynaddsubfx_gui ${UI_fl_files})

add_library(zynaddsubfx_gui STATIC
    ${zynaddsubfx_gui_FLTK_UI_SRCS}
    NioUI.cpp
    WidgetPDial.cpp)

if(LibloEnable)
    target_sources(zynaddsubfx_gui
        PRIVATE NSM.C
        PRIVATE NSM/Client.C)
endif()

target_include_directories(zynaddsubfx_gui
    PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/../MXml"
    PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

target_compile_definitions(zynaddsubfx_gui
    PRIVATE -DPIXMAP_PATH="${CMAKE_INSTALL_PREFIX}/share/zynaddsubfx/pixmaps/")

target_compile_definitions(zynaddsubfx_gui
    PRIVATE -DSOURCE_DIR="${CMAKE_CURRENT_SOURCE_DIR}")

if(NtkGui)
  target_link_libraries(zynaddsubfx_gui
      PRIVATE ${NTK_LIBRARIES})
endif(NtkGui)

if(FltkGui)
  target_link_libraries(zynaddsubfx_gui
      PRIVATE ${FLTK_LIBRARIES})
endif(FltkGui)
