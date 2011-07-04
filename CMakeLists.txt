# build:
# mkdir build
# cd build
# cmake .. -G "MinGW Makefiles"

cmake_minimum_required(VERSION 2.8)
project ( hunspell )
SET(HUNSPELL_VERSION 1.3.2)

set(CPACK_GENERATOR "TBZ2")
include(CPack)

if(WIN32)
    add_definitions( -DWIN32 -DMINGW32 -c -O2 -frtti -fexceptions -mthreads -enable-stdcall-fixup -Wl,-enable-auto-import -Wl,-enable-runtime-pseudo-reloc)
endif(WIN32)

INCLUDE_DIRECTORIES (./ src/hunspell src/parsers src/tools src/win_api)


# LIBS
SET(SRCS
    src/hunspell/affentry.cxx
    src/hunspell/affixmgr.cxx
    src/hunspell/csutil.cxx
    src/hunspell/dictmgr.cxx
    src/hunspell/hashmgr.cxx
    src/hunspell/suggestmgr.cxx
    src/hunspell/license.myspell
    src/hunspell/license.hunspell
    src/hunspell/phonet.cxx
    src/hunspell/filemgr.cxx
    src/hunspell/hunzip.cxx
    src/hunspell/hunspell.cxx
    src/hunspell/replist.cxx)

SET(HEADERS
    src/hunspell/affentry.hxx
    src/hunspell/htypes.hxx
    src/hunspell/affixmgr.hxx
    src/hunspell/csutil.hxx
    src/hunspell/hunspell.hxx
    src/hunspell/atypes.hxx
    src/hunspell/dictmgr.hxx
    src/hunspell/hunspell.h
    src/hunspell/suggestmgr.hxx
    src/hunspell/baseaffix.hxx
    src/hunspell/hashmgr.hxx
    src/hunspell/langnum.hxx
    src/hunspell/phonet.hxx
    src/hunspell/filemgr.hxx
    src/hunspell/hunzip.hxx
    src/hunspell/w_char.hxx
    src/hunspell/replist.hxx
    src/hunspell/hunvisapi.h)

ADD_LIBRARY( hunspell-static STATIC ${HEADERS} ${SRCS})
TARGET_LINK_LIBRARIES( hunspell-static  )
ADD_DEPENDENCIES( hunspell-static  ${HEADERS})
INSTALL(TARGETS hunspell-static RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)

ADD_LIBRARY( hunspell SHARED ${HEADERS} ${SRCS})
# set Windows DLL version information 
IF(WIN32)
      ADD_CUSTOM_COMMAND(TARGET hunspell PRE_BUILD  COMMAND windres -i ${CMAKE_CURRENT_SOURCE_DIR}/src/win_api/Hunspell.rc -o Hunspell_res.o COMMENT "Windows DLL version")
      SET_TARGET_PROPERTIES(hunspell PROPERTIES LINK_FLAGS "Hunspell_res.o")
ENDIF(WIN32)
SET_TARGET_PROPERTIES( hunspell PROPERTIES COMPILE_FLAGS "-DMAKE_DLL")
ADD_DEPENDENCIES( hunspell ${HEADERS})
TARGET_LINK_LIBRARIES( hunspell )
INSTALL(TARGETS hunspell RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)

SET_TARGET_PROPERTIES(
    hunspell
    hunspell-static
  PROPERTIES
    VERSION ${HUNSPELL_VERSION}
    SOVERSION ${HUNSPELL_VERSION}
    PREFIX "lib"
    OUTPUT_NAME "hunspell")

# install header files
install(FILES ${HEADERS} DESTINATION include)

SET(PARSER_SRCS
    src/parsers/firstparser.cxx
    src/parsers/htmlparser.cxx
    src/parsers/latexparser.cxx
    src/parsers/manparser.cxx
    src/parsers/textparser.cxx)

SET(PARSER_HEADERS
    src/parsers/firstparser.hxx
    src/parsers/htmlparser.hxx
    src/parsers/manparser.hxx
    src/parsers/latexparser.hxx
    src/parsers/textparser.hxx)

# install header files
install(FILES ${PARSER_HEADERS} DESTINATION include)

SET(DOC
    README.md
    README
    README.myspell
    AUTHORS
    AUTHORS.myspell
    THANKS
    BUGS
    ChangeLog
    ChangeLog.O
    COPYING
    COPYING.LGPL
    COPYING.MPL)
    
SET(MAN
    man/hunspell.1
    man/hunspell.3
    man/hunspell.4
    man/hzip.1
    man/hunzip.1)

# install header files
install(FILES ${DOC} DESTINATION doc)
install(FILES ${MAN} DESTINATION doc/man)

ADD_LIBRARY( parsers-static STATIC ${PARSER_HEADERS} ${PARSER_SRCS})
SET_TARGET_PROPERTIES( hunspell-static PROPERTIES PREFIX "lib")
TARGET_LINK_LIBRARIES( parsers-static hunspell-static )
ADD_DEPENDENCIES( parsers-static ${PARSER_HEADERS})
INSTALL(TARGETS parsers-static RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_LIBRARY( parsers SHARED ${PARSER_HEADERS} ${PARSER_SRCS})
SET_TARGET_PROPERTIES( parsers PROPERTIES COMPILE_FLAGS "-DMAKE_DLL")
SET_TARGET_PROPERTIES( hunspell-static PROPERTIES PREFIX "lib")
TARGET_LINK_LIBRARIES( parsers hunspell)
ADD_DEPENDENCIES( parsers ${PARSER_HEADERS})
INSTALL(TARGETS parsers RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)

# programs
ADD_EXECUTABLE(hunspell_exe src/tools/hunspell.cxx)
SET_TARGET_PROPERTIES(hunspell_exe PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
SET_TARGET_PROPERTIES(hunspell_exe PROPERTIES OUTPUT_NAME "hunspell")
TARGET_LINK_LIBRARIES(hunspell_exe parsers hunspell)
INSTALL(TARGETS hunspell_exe RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(testparser ${PARSER_SRCS} src/parsers/testparser.cxx)
SET_TARGET_PROPERTIES(testparser PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(testparser hunspell)
INSTALL(TARGETS testparser RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(hzip src/tools/hzip.c)
SET_TARGET_PROPERTIES(hzip PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(hzip)
INSTALL(TARGETS hzip RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(hunzip src/tools/hunzip.cxx)
SET_TARGET_PROPERTIES(hunzip PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(hunzip hunspell)
INSTALL(TARGETS hunzip RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(munch src/tools/munch.c src/tools/munch.h)
SET_TARGET_PROPERTIES(munch PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(munch)
INSTALL(TARGETS munch RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(unmunch src/tools/unmunch.c src/tools/unmunch.h)
SET_TARGET_PROPERTIES(unmunch PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(unmunch)
INSTALL(TARGETS unmunch RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(analyze src/tools/analyze.cxx)
SET_TARGET_PROPERTIES(analyze PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(analyze hunspell)
INSTALL(TARGETS analyze RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)
                                 
ADD_EXECUTABLE(chmorph src/tools/chmorph.cxx)
SET_TARGET_PROPERTIES(chmorph PROPERTIES COMPILE_FLAGS "-DMAKE_EXE")
TARGET_LINK_LIBRARIES(chmorph parsers hunspell)
INSTALL(TARGETS chmorph RUNTIME DESTINATION bin
                                 LIBRARY DESTINATION lib
                                 ARCHIVE DESTINATION lib)