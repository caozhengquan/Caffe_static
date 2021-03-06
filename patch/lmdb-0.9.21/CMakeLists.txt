# cmake project script for lmdb 0.9.21
# author guyadong@gdface.net 2017/06/22
CMAKE_MINIMUM_REQUIRED(VERSION 3.0)
PROJECT(lmdb VERSION  0.9.21 LANGUAGES  C)
OPTION(CLOSE_WARNING "close warning,default off" OFF)
OPTION(BUILD_TEST "build test program,default off" OFF)
IF (MSVC)
	# for #include <unistd.h>
	INCLUDE_DIRECTORIES( ${PROJECT_SOURCE_DIR} )
ENDIF(MSVC)
# option 
IF( CLOSE_WARNING)
	#close warning
	IF (MSVC)		
		ADD_COMPILE_OPTIONS( /wd4267 /wd4996 /wd4477 /wd4244 /wd4146 /wd4333 /wd4172)	
		ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
	ELSE()	
		ADD_COMPILE_OPTIONS(-Wno-return-local-addr)	
	ENDIF(MSVC)
ENDIF(CLOSE_WARNING)

# library
# use BUILD_SHARED_LIBS to control STATIC OR SHARED library generated
ADD_LIBRARY(lmdb mdb.c midl.c)
IF (UNIX)
	SET(CMAKE_EXE_LINKER_FLAGS "-pthread ${CMAKE_EXE_LINKER_FLAGS}")
	SET(CMAKE_SHARED_LINKER_FLAGS "-pthread ${CMAKE_SHARED_LINKER_FLAGS}")
ENDIF (UNIX)

# add executable target 
FUNCTION (LMDB_ADD_EXE EXE_NAME SOURCE_FILE)
    IF (NOT EXE_NAME)
        MESSAGE(FATAL_ERROR "Error: LMDB_ADD_TEST called without test name")
    ENDIF(NOT EXE_NAME)

    IF (NOT SOURCE_FILE)
        MESSAGE(FATAL_ERROR "Error: LMDB_ADD_TEST called without test file")
    ENDIF(NOT SOURCE_FILE)
		IF (${SOURCE_FILE})
	    ADD_EXECUTABLE(${EXE_NAME} ${${SOURCE_FILE}})
		ELSE()
	    ADD_EXECUTABLE(${EXE_NAME} ${SOURCE_FILE})
		ENDIF()

    TARGET_LINK_LIBRARIES(${EXE_NAME} lmdb)

    #ADD_TEST(NAME ${EXE_NAME} COMMAND ${EXE_NAME})

    #ADD_DEPENDENCIES(RUN_LMDB_UNIT_TESTS ${EXE_NAME})
    INSTALL (TARGETS ${EXE_NAME}  RUNTIME DESTINATION bin)
ENDFUNCTION (LMDB_ADD_EXE)
IF (MSVC)
	# add getopt.c for #include <unistd.h> 
	SET(MDB_STAT_SRCS mdb_stat.c getopt.c)
	SET(MDB_COPY_SRCS mdb_copy.c getopt.c)
	SET(MDB_DUMP_SRCS mdb_dump.c getopt.c)
	SET(MDB_LOAD_SRCS mdb_load.c getopt.c)
ELSE()
	SET(MDB_STAT_SRCS mdb_stat.c)
	SET(MDB_COPY_SRCS mdb_copy.c)
	SET(MDB_DUMP_SRCS mdb_dump.c)
	SET(MDB_LOAD_SRCS mdb_load.c)
ENDIF(MSVC)   
            
LMDB_ADD_EXE(mdb_stat MDB_STAT_SRCS)
LMDB_ADD_EXE(mdb_copy MDB_COPY_SRCS)
LMDB_ADD_EXE(mdb_dump MDB_DUMP_SRCS)
LMDB_ADD_EXE(mdb_load MDB_LOAD_SRCS)
IF ( BUILD_TEST )
	LMDB_ADD_EXE(mtest mtest.c)
	LMDB_ADD_EXE(mtest2 mtest2.c)
	LMDB_ADD_EXE(mtest3 mtest3.c)
	LMDB_ADD_EXE(mtest4 mtest4.c)
	LMDB_ADD_EXE(mtest5 mtest5.c)
ENDIF(BUILD_TEST)

INSTALL (TARGETS lmdb mdb_stat mdb_copy mdb_dump mdb_load 
	RUNTIME DESTINATION bin 
	LIBRARY DESTINATION lib 
	ARCHIVE DESTINATION lib)
INSTALL (FILES lmdb.h DESTINATION include)
INSTALL (FILES  mdb_stat.1 mdb_copy.1 mdb_dump.1 mdb_load.1 	DESTINATION share/man/man1)

