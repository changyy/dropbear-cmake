cmake_minimum_required(VERSION 2.8)
project (dropbear-sshd)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

message("")
if (NOT DEFINED _PATH_SSH_PROGRAM)
	set(_PATH_SSH_PROGRAM "/tmp/bin/dbclient")
endif (NOT DEFINED _PATH_SSH_PROGRAM)
message("[_PATH_SSH_PROGRAM] (use -D_PATH_SSH_PROGRAM=/path/dbclient to redefine ) : ${_PATH_SSH_PROGRAM}")
if (NOT DEFINED DEFAULT_PATH)
	set(DEFAULT_PATH "/usr/bin:/bin:/tmp/bin")
endif (NOT DEFINED DEFAULT_PATH)
message("[DEFAULT_PATH] (use -DDEFAULT_PATH=/path/bin:/path2/bin to redefine ) : ${DEFAULT_PATH}")
message("")

set(TOOL_DROPBEAR_PATCH dropbear-2013.56.patch)
set(TOOL_DROPBEAR_SOURCE dropbear-2013.56.tar.bz2)
set(TOOL_DROPBEAR_DIR dropbear-2013.56)
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_DROPBEAR
			COMMAND curl https://matt.ucc.asn.au/dropbear/${TOOL_DROPBEAR_SOURCE} | tar -xvjf - && cd ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_DROPBEAR_PATCH} && ${target_platform_preconfigure_options} ./configure --host=${target_platform} --disable-zlib && CFLAGS=' -D_PATH_SSH_PROGRAM=\\"${_PATH_SSH_PROGRAM}\\" -DDEFAULT_PATH=\\"${DEFAULT_PATH}\\" ' make -j2 && make scp && mkdir -p ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} && cp ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR}/dbclient ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR}/dropbear ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR}/scp ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
			WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
			COMMENT "Build dropbear"
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_DROPBEAR
			COMMAND curl https://matt.ucc.asn.au/dropbear/${TOOL_DROPBEAR_SOURCE} | tar -xvjf - && cd ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_DROPBEAR_PATCH} && ./configure && CFLAGS=' -D_PATH_SSH_PROGRAM=\\"${_PATH_SSH_PROGRAM}\\" -DDEFAULT_PATH=\\"${DEFAULT_PATH}\\" ' make -j2 && CFLAGS=' -D_PATH_SSH_PROGRAM=\\"${_PATH_SSH_PROGRAM}\\" -DDEFAULT_PATH=\\"${DEFAULT_PATH}\\" ' make scp && mkdir -p ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} && cp ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR}/dbclient ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR}/dropbear ${CMAKE_BINARY_DIR}/${TOOL_DROPBEAR_DIR}/scp ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
			WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
			COMMENT "Build dropbear" 
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(build_dropbear ALL DEPENDS BUILD_DROPBEAR )
