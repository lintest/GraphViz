BISON_TARGET(HTMLparse htmlparse.y ${CMAKE_CURRENT_BINARY_DIR}/htmlparse.c)

if(BUILD_SHARED_LIBS)
    add_definitions(-DGVC_EXPORTS -D_BLD_gvc=1)
endif()    

# Generate colortbl.h from sources
add_custom_command(
    OUTPUT svgcolor_lib
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/svgcolor_names ${TOP_SOURCE_DIR}/awk/svgcolor.awk
    COMMAND ${AWK_EXECUTABLE} -f ${TOP_SOURCE_DIR}/awk/svgcolor.awk
            ${CMAKE_CURRENT_SOURCE_DIR}/svgcolor_names > ${CMAKE_CURRENT_BINARY_DIR}/svgcolor_lib
)
add_custom_command(
    OUTPUT brewer_lib
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/brewer_colors ${TOP_SOURCE_DIR}/awk/brewer.awk
    COMMAND ${AWK_EXECUTABLE} -f ${TOP_SOURCE_DIR}/awk/brewer.awk
            ${CMAKE_CURRENT_SOURCE_DIR}/brewer_colors > ${CMAKE_CURRENT_BINARY_DIR}/brewer_lib
)
configure_file(
    "${TOP_SOURCE_DIR}/cmake/generate_color_lib.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/generate_color_lib.cmake"
    @ONLY
)
add_custom_command(
    OUTPUT color_lib
    DEPENDS svgcolor_lib brewer_lib ${CMAKE_CURRENT_SOURCE_DIR}/color_names
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/generate_color_lib.cmake
)
add_custom_command(
    OUTPUT common/colortbl.h
    DEPENDS color_lib
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/common"
    COMMAND ${AWK_EXECUTABLE} -f ${TOP_SOURCE_DIR}/awk/colortbl.awk
            ${CMAKE_CURRENT_BINARY_DIR}/color_lib > ${CMAKE_CURRENT_BINARY_DIR}/common/colortbl.h
)

# Generate ps_font_equiv.h from sources
add_custom_command(
    OUTPUT ps_font_equiv.h
    DEPENDS
        ${CMAKE_CURRENT_SOURCE_DIR}/mksvgfonts.pl
        ${CMAKE_CURRENT_SOURCE_DIR}/ps_font_equiv.txt
        ${CMAKE_CURRENT_SOURCE_DIR}/fontmap.cfg
        ${CMAKE_CURRENT_SOURCE_DIR}/ps_fontmap.txt
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/ps_fontmap.txt
                                     ${CMAKE_CURRENT_BINARY_DIR}/ps_font_equiv.h
    COMMAND ${PERL_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/mksvgfonts.pl
                               ${CMAKE_CURRENT_SOURCE_DIR}/fontmap.cfg
                               ${CMAKE_CURRENT_SOURCE_DIR}/ps_font_equiv.txt
                               >> ${CMAKE_CURRENT_BINARY_DIR}/ps_font_equiv.h
)

add_library(common_obj OBJECT
    # Header files
    arith.h
    color.h
    colorprocs.h
    ${CMAKE_CURRENT_BINARY_DIR}/common/colortbl.h
    const.h
    entities.h
    geom.h
    geomprocs.h
    globals.h
    intset.h
    htmllex.h
    htmltable.h
    logic.h
    macros.h
    memory.h
    pointset.h
    ps_font_equiv.h
    render.h
    textspan.h
    types.h
    usershape.h
    utils.h

    # Source files
    args.c
    arrows.c
    colxlate.c
    ellipse.c
    emit.c
    geom.c
    globals.c
    htmllex.c
    htmltable.c
    input.c
    intset.c
    labels.c
    ns.c
    memory.c
    output.c
    pointset.c
    postproc.c
    psusershape.c
    routespl.c
    shapes.c
    splines.c
    taper.c
    textspan.c
    timing.c
    utils.c

    # Generated files
    ${BISON_HTMLparse_OUTPUTS}
)

target_include_directories(common_obj PRIVATE
    ${GRAPHVIZ_LIB_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${GRAPHVIZ_LIB_DIR}/cdt
    ${GRAPHVIZ_LIB_DIR}/cgraph
    ${GRAPHVIZ_LIB_DIR}/gvc
    ${GRAPHVIZ_LIB_DIR}/pack
    ${GRAPHVIZ_LIB_DIR}/pathplan
    ${GRAPHVIZ_LIB_DIR}/xdot
)

if (WIN32)
    target_include_directories(common_obj PRIVATE ${WINDOWS_DEPENDENCY_DIR}/include)
endif()

# From https://gitlab.com/graphviz/graphviz/-/issues/1613 @mdwies 20093010
if(APPLE)
    target_compile_options(common_obj PRIVATE "-fno-common")
endif(APPLE)

add_library(common STATIC
    $<TARGET_OBJECTS:common_obj>
)

# Specify headers to be installed
install(
    FILES
        arith.h
        color.h
        geom.h
        textspan.h
        types.h
        usershape.h
    DESTINATION ${HEADER_INSTALL_DIR}
)
