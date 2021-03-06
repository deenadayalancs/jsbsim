include(${QT_USE_FILE})
add_definitions(-DUSE_QT4)

if (WITH_ARKOSG AND WITH_PREFER_STATIC)
    add_definitions(-DOSG_LIBRARY_STATIC)
endif()

link_directories(
    ${OPENSCENEGRAPH_LIBRARY_DIRS}
    ${QT_LIBRARY_DIRS}
    )

include_directories(
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_BINARY_DIR}
    ${CMAKE_BINARY_DIR}/gui
    ${ARKOSG_INCLUDE_DIRS}
    ${OPENSCENEGRAPH_INCLUDE_DIRS}
    ${QT_INCLUDE_DIRS}
    ${OPENGL_INCLUDE_DIR}
    ${Boost_INCLUDE_DIRS}
    )

set(JSBSIMGUI_MOC_SRC
    MainWindow.hpp
    )
set(JSBSIMGUI_UI_SRC
    )
if (WITH_ARKOSG)
    list(APPEND JSBSIMGUI_UI_SRC
        MainWindow_OSG.ui
        )
else()
    list(APPEND JSBSIMGUI_UI_SRC
        MainWindow.ui
        )
endif()
#set(JSBSIMGUI_RES_SRC
    #textures.qrc
    #)

qt4_wrap_cpp(JSBSIMGUI_MOC ${JSBSIMGUI_MOC_SRC})
qt4_wrap_ui(JSBSIMGUI_UI ${JSBSIMGUI_UI_SRC})
#qt4_add_resources(JSBSIMGUI_RES ${JSBSIMGUI_RES_SRC})

set(JSBSIMGUI_SRCS
    MainWindow.cpp
    main.cpp
    ${JSBSIMGUI_MOC}
    ${JSBSIMGUI_UI}
    #${JSBSIMGUI_RES}
    )

# set icns file containing icons
if (APPLE)
    # set how it shows up in Info.plist (no prepended path!!)
    set(MACOSX_BUNDLE_ICON_FILE jsbsim.icns)
    # set where in the bundle to put the icns file
    set_source_files_properties(${CMAKE_SOURCE_DIR}/cmake/jsbsim.icns PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    # include the icns file in the target
    list(APPEND JSBSIMGUI_SRCS ${CMAKE_SOURCE_DIR}/cmake/jsbsim.icns)
endif()

# resources
install(FILES plane.ac jsbsim.png DESTINATION share/jsbsim/gui COMPONENT Runtime)

# install gnome menu
if (UNIX AND NOT APPLE)
    install(FILES jsbsim.desktop DESTINATION share/applications COMPONENT Runtime)
endif()

# linking
set(JSBSIMGUI_LIBS
    jsbsimStatic
    ${QT_LIBRARIES}
    )
if(MINGW)
    list(APPEND JSBSIMGUI_LIBS lcms lzma)
endif()
if (MSVC)
    add_executable(jsbsim-gui ${JSBSIMGUI_SRCS})
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /FORCE:Multiple")
else()
    add_executable(jsbsim-gui WIN32 MACOSX_BUNDLE ${JSBSIMGUI_SRCS})
endif()
if (WITH_ARKOSG)
    list(APPEND JSBSIMGUI_LIBS
        ${ARKOSG_LIBRARIES}
        ${OSGPLUGIN_LIBRARIES}
        ${OPENSCENEGRAPH_LIBRARIES}
        ${OPENGL_LIBRARIES}
        )
    if(UNIX AND NOT APPLE)
        list(APPEND JSBSIMGUI_LIBS Xrandr)
    endif()
    add_dependencies(jsbsim-gui arkosg qt osg)
endif()
target_link_libraries(jsbsim-gui ${JSBSIMGUI_LIBS})
message(STATUS "installed targets: ${INSTALLED_TARGETS}")
install(TARGETS jsbsim-gui 
    RUNTIME DESTINATION "bin" COMPONENT Runtime
    BUNDLE DESTINATION "/Applications" COMPONENT Runtime
    )

if(APPLE)
    add_custom_command(TARGET jsbsim-gui POST_BUILD COMMAND macdeployqt ${CMAKE_BINARY_DIR}/gui/jsbsim-gui.app)
    add_custom_command(TARGET jsbsim-gui POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory /opt/local/lib/Resources/qt_menu.nib ${CMAKE_BINARY_DIR}/gui/jsbsim-gui.app/Contents/Resources/qt_menu.nib)
endif()
