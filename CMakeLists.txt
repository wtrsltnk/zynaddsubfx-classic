cmake_minimum_required(VERSION 3.8)

project(zynaddsubfx)

set(VERSION "2.4.4")

enable_testing()
include(CTestConfig.cmake)

add_subdirectory(src)

if(CompileTests)
    add_subdirectory(tests)
endif(CompileTests)

install(FILES AUTHORS.txt COPYING FAQ.txt HISTORY.txt README.txt
    DESTINATION share/doc/zynaddsubfx
    )

install(FILES zynaddsubfx-jack.desktop zynaddsubfx-alsa.desktop
    DESTINATION share/applications)

install(FILES zynaddsubfx.svg
    DESTINATION share/pixmaps)

install(DIRECTORY instruments/banks
    DESTINATION share/zynaddsubfx)

install(DIRECTORY instruments/examples
    DESTINATION share/zynaddsubfx)
