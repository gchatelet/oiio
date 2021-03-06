if (NOT BOOST_CUSTOM)
    find_package (PythonLibs ${PYTHON_VERSION} REQUIRED)
    find_package (Boost 1.42 REQUIRED COMPONENTS python)
else ()
    find_package (PythonLibs ${PYTHON_VERSION} REQUIRED)
endif ()

if (APPLE)
#    set (PYTHON_LIBRARIES /opt/local/lib)
endif ()

# Disable some warnings for Clang, it's a little too picky with boost
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_definitions ("-Wno-array-bounds")
endif ()

if (BOOST_CUSTOM OR Boost_FOUND AND PYTHONLIBS_FOUND)

    set (python_srcs py_imageinput.cpp py_imageoutput.cpp
         py_imagecache.cpp py_imagespec.cpp py_roi.cpp
         py_imagebuf.cpp py_imagebufalgo.cpp
         py_typedesc.cpp py_paramvalue.cpp
         py_oiio.cpp)

    message (STATUS "Python found ${PYTHONLIBS_FOUND} ")
    message (STATUS "Python include dirs ${PYTHON_INCLUDE_PATH}")
    message (STATUS "Python libraries    ${PYTHON_LIBRARIES}")
    message (STATUS "Python to include 'lib' prefix: ${PYLIB_LIB_PREFIX}")
    message (STATUS "Python to include SO version: ${PYLIB_INCLUDE_SONAME}")

    include_directories (${PYTHON_INCLUDE_PATH} ${Boost_INCLUDE_DIRS})
    add_library (PyOpenImageIO SHARED ${python_srcs})
    target_link_libraries (PyOpenImageIO OpenImageIO ${Boost_LIBRARIES} ${Boost_Python_LIBRARIES} ${PYTHON_LIBRARIES} ${CMAKE_DL_LIBS})

    # Exclude the 'lib' prefix from the name
    if(NOT PYLIB_LIB_PREFIX)
        add_definitions("-DOIIO_PYMODULE_NAME=OpenImageIO")
        set_target_properties (PyOpenImageIO PROPERTIES
                                 OUTPUT_NAME OpenImageIO
                                 PREFIX "")
    else()
        add_definitions("-DOIIO_PYMODULE_NAME=PyOpenImageIO")
        set_target_properties (PyOpenImageIO PROPERTIES
                                 OUTPUT_NAME PyOpenImageIO
                                 PREFIX lib)
    endif ()

    if(PYLIB_INCLUDE_SONAME)
        message(STATUS "Setting PyOIIO SOVERSION to: ${SOVERSION}")
        set_target_properties(PyOpenImageIO PROPERTIES
            VERSION ${OIIO_VERSION_MAJOR}.${OIIO_VERSION_MINOR}
            SOVERSION ${SOVERSION}
        )
    endif()

    if (APPLE)
        # Python seems to only look for plugins that end in .so, not .dylib
        set_target_properties (PyOpenImageIO PROPERTIES SUFFIX ".so")
    endif ()

    if (WIN32)
        set_target_properties (PyOpenImageIO PROPERTIES
                               DEBUG_POSTFIX "_d"
                               SUFFIX ".pyd")
    endif()
    
    install (TARGETS PyOpenImageIO
             RUNTIME DESTINATION ${PYLIB_INSTALL_DIR} COMPONENT user
             LIBRARY DESTINATION ${PYLIB_INSTALL_DIR} COMPONENT user)

endif ()
