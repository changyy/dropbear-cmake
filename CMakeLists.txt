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

set(TOOL_DROPBEAR_ENABLE_PATH_REDEFINE_PATCH dropbear-2013.58-enable-path-redefine-path.patch)
set(TOOL_DROPBEAR_PAM_NONEXISTENT_USER_HANDLE_PATCH dropbear-2013.58-pam-nonexistent-user-handle.patch)
set(TOOL_DROPBEAR_SOURCE dropbear-2013.58.tar.bz2)
set(TOOL_DROPBEAR_DIR dropbear-2013.58)

set(OTHER_CONFIGURE "")
set(SELF_CPPFLAGS "")
set(SELF_CFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

message("")
if (NOT DEFINED _PATH_SSH_PROGRAM)
	set(_PATH_SSH_PROGRAM "/tmp/bin/dbclient")
endif (NOT DEFINED _PATH_SSH_PROGRAM)
message("[_PATH_SSH_PROGRAM] (use -D_PATH_SSH_PROGRAM=/path/dbclient to redefine ) : ${_PATH_SSH_PROGRAM}")
if (NOT DEFINED DEFAULT_PATH)
	set(DEFAULT_PATH "/usr/bin:/bin:/tmp/bin")
endif (NOT DEFINED DEFAULT_PATH)
message("[DEFAULT_PATH] (use -DDEFAULT_PATH=/path/bin:/path2/bin to redefine ) : ${DEFAULT_PATH}")
if (NOT DEFINED PAM_ENABLE)
	message("[PAM_ENABLE] (use -DPAM_ENABLE:bool=ON to enable PAM usage)")
else(NOT DEFINED PAM_ENABLE)
message("[PAM_ENABLE] (remove -DPAM_ENABLE:bool=ON to disable PAM usage) : ${PAM_ENABLE}")
	set(OTHER_CONFIGURE "${OTHER_CONFIGURE};--enable-pam")
	if(NOT TARGET build_libpam)
		add_subdirectory(libpam-cmake)
	endif(NOT TARGET build_libpam)
	set(SELF_CFLAGS "${SELF_CFLAGS};-I${CMAKE_BINARY_DIR}/include")
	set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_BINARY_DIR}/include")
	set(SELF_LDFLAGS "${SELF_LDFLAGS};-L${CMAKE_BINARY_DIR}/lib")
	set(SELF_DEPENDS "${SELF_DEPENDS};build_libpam")
endif (NOT DEFINED PAM_ENABLE)
message("")

	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_DROPBEAR
			# get source
			COMMAND curl https://matt.ucc.asn.au/dropbear/releases/${TOOL_DROPBEAR_SOURCE} | tar -xvjf - 
			# patch
			COMMAND cd ${TOOL_DROPBEAR_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_DROPBEAR_PAM_NONEXISTENT_USER_HANDLE_PATCH} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_DROPBEAR_ENABLE_PATH_REDEFINE_PATCH} 
			# configure
			COMMAND cd ${TOOL_DROPBEAR_DIR} && CFLAGS='${SELF_CFLAGS} -D_PATH_SSH_PROGRAM=\\"${_PATH_SSH_PROGRAM}\\" -DDDEFAULT_PATH=\\"${DEFAULT_PATH}\\" ' CPPFLAGS="${SELF_CPPFLAGS}" LDFLAGS="${SELF_LDFLAGS}" ${target_platform_preconfigure_options} ./configure --host=${target_platform} --disable-zlib ${OTHER_CONFIGURE}
			# build
			COMMAND cd ${TOOL_DROPBEAR_DIR} && make -j4 && make scp
			# finish
			COMMAND cd ${TOOL_DROPBEAR_DIR} && mkdir -p ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} && cp dbclient dropbear scp ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT "Build dropbear"
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_DROPBEAR
			COMMAND curl https://matt.ucc.asn.au/dropbear/releases/${TOOL_DROPBEAR_SOURCE} | tar -xvjf - 
			COMMAND cd ${TOOL_DROPBEAR_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_DROPBEAR_PAM_NONEXISTENT_USER_HANDLE_PATCH} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_DROPBEAR_ENABLE_PATH_REDEFINE_PATCH}
			COMMAND cd ${TOOL_DROPBEAR_DIR} && CFLAGS='${SELF_CFLAGS} -D_PATH_SSH_PROGRAM=\\"${_PATH_SSH_PROGRAM}\\" -DDDEFAULT_PATH=\\"${DEFAULT_PATH}\\" ' CPPFLAGS="${SELF_CPPFLAGS}" LDFLAGS="${SELF_LDFLAGS}" ./configure ${OTHER_CONFIGURE} 
			COMMAND cd ${TOOL_DROPBEAR_DIR} && make -j4 && make scp 
			COMMAND cd ${TOOL_DROPBEAR_DIR} && mkdir -p ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} && cp dbclient dropbear scp ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT "Build dropbear" 
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(build_dropbear ALL DEPENDS ${SELF_DEPENDS} BUILD_DROPBEAR )
