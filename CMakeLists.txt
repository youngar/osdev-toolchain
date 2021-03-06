cmake_minimum_required(VERSION 3.2)

project(osdev-toolchain VERSION 1.0.0 LANGUAGES C CXX)

#setup various directory config options
set(TOOLCHAIN_DIST_DIR "${CMAKE_CURRENT_BINARY_DIR}/src/dist" CACHE PATH "Directory where toolchain source tarballs will be downloaded")
set(TOOLCHAIN_SRC_ROOT "${CMAKE_CURRENT_BINARY_DIR}/src" CACHE PATH "Directory where toochain sources will be extracted")
set(TOOLCHAIN_BUILD_ROOT "${CMAKE_CURRENT_BINARY_DIR}/src/build" CACHE PATH "Directory where toolchains will be built")

set(TOOLCHAIN_STAMP_ROOT "${TOOLCHAIN_SRC_ROOT}" CACHE PATH  "Directory where timestamps are put")
set(TOOLCHAIN_TMP_DIR "${CMAKE_BINARY_DIR}/tmp" CACHE PATH "Directory where temp stuff is put")

set(TOOLCHAIN_INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR} CACHE PATH "Directory where toolchain binaries will be installed")

set(TOOLCHAIN_TARGET i686-elf CACHE STRING "What architecture you want to target")


include(ExternalProject)

set_property(DIRECTORY PROPERTY EP_STEP_TARGETS configure build)

set(TOOLCHAIN_BINUTILS_VERSION 2.27)
set(TOOLCHAIN_BINUTILS_HASH 41b053ed4fb2c6a8173ef421460fbb28)
set(TOOLCHAIN_BINUTILS_URL ftp://ftp.gnu.org/gnu/binutils/binutils-${TOOLCHAIN_BINUTILS_VERSION}.tar.gz)


message(STATUS "TMPDIR = " ${TOOLCHAIN_TMP_DIR})


message(STATUS "StampDIR = " ${TOOLCHAIN_STAMP_ROOT}/binutils-${TOOLCHAIN_BINUTILS_VERSION}-stamp)
message(STATUS "Downloa dDIR = " ${TOOLCHAIN_DIST_DIR})
message(STATUS "Src Dir = " ${TOOLCHAIN_SRC_ROOT}/binutils-${TOOLCHAIN_BINUTILS_VERSION})
message(STATUS "InstallDir = " ${TOOLCHAIN_INSTALL_DIR})


find_program(MAKEINFO_PATH makeinfo)
if(NOT MAKEINFO_PATH)
	message(FATAL_ERROR "makeinfo executable not found. Try installing texinfo package")
endif()

find_program(FLEX_PATH flex)
if(NOT FLEX_PATH)
	message(FATAL_ERROR "flex is required to build binutils/gcc")
endif()

find_program(BISON_PATH bison)
if(NOT BISON_PATH)
	message(FATAL_ERROR "Bison is required to build binutils/gcc")
endif()

ExternalProject_Add( binutils
	URL ${TOOLCHAIN_BINUTILS_URL}
	URL_HASH MD5=${TOOLCHAIN_BINUTILS_HASH}

	#set up our directories
	TMP_DIR ${TOOLCHAIN_TMP_DIR}
	STAMP_DIR ${TOOLCHAIN_STAMP_ROOT}/binutils-${TOOLCHAIN_BINUTILS_VERSION}-stamp
	DOWNLOAD_DIR ${TOOLCHAIN_DIST_DIR}
	SOURCE_DIR ${TOOLCHAIN_SRC_ROOT}/binutils-${TOOLCHAIN_BINUTILS_VERSION}
	BINARY_DIR ${TOOLCHAIN_BUILD_ROOT}/binutils-${TOOLCHAIN_BINUTILS_VERSION}-${TOOLCHAIN_TARGET}
	INSTALL_DIR ${TOOLCHAIN_INSTALL_DIR}



	CONFIGURE_COMMAND  ${TOOLCHAIN_SRC_ROOT}/binutils-${TOOLCHAIN_BINUTILS_VERSION}/configure --prefix=${TOOLCHAIN_INSTALL_DIR} --target=${TOOLCHAIN_TARGET}  --with-sysroot --disable-nls --disable-werror


	BUILD_COMMAND make
)



set(GCC_VERSION 6.3.0)
set(GCC_HASH MD5=6e5ea04789678f1250c1b30c4d9ec417)
set(GCC_URL ftp://ftp.gnu.org/gnu/gcc/gcc-${GCC_VERSION}/gcc-${GCC_VERSION}.tar.gz)


#we need to call configure with a relative path, otherwise it breaks windows builds
file(RELATIVE_PATH GCC_CONFIG_PATH ${TOOLCHAIN_BUILD_ROOT}/gcc  ${TOOLING_SRC_DIR}/gcc/configure)

ExternalProject_Add( gcc
	DEPENDS binutils
	URL ${GCC_URL}
	URL_HASH ${GCC_HASH}

	DEPENDS binutils
	#set up our directories
	TMP_DIR ${TOOLCHAIN_TMP_DIR}
	STAMP_DIR ${TOOLCHAIN_STAMP_ROOT}/gcc-stamp
	DOWNLOAD_DIR ${TOOLCHAIN_DIST_DIR}
	SOURCE_DIR ${TOOLCHAIN_SRC_ROOT}/gcc-${GCC_VERSION}
	BINARY_DIR ${TOOLCHAIN_BUILD_ROOT}/gcc-${GCC_VERSION}-${TOOLCHAIN_TARGET}
	INSTALL_DIR ${TOOLCHAIN_INSTALL_DIR}


	CONFIGURE_COMMAND ${GCC_CONFIG_PATH} --prefix=$<SHELL_PATH:${TOOLCHAIN_PREFIX}> --target=${TOOLCHAIN_TARGET}  --disable-nls  --enable-languages=c,c++ --without-headers
	BUILD_COMMAND \$(MAKE) all-gcc all-target-libgcc
	INSTALL_COMMAND make install-gcc install-target-libgcc
)

set(TOOLCHAIN_NASM_VERSION 2.12.02)
set(NASM_HASH MD5=c75b78faf2d3797b431047f9c8656c09)

set(NASM_URL http://www.nasm.us/pub/nasm/releasebuilds/${NASM_VERSION}/nasm-${NASM_VERSION}.tar.gz)

ExternalProject_Add(nasm
	URL ${NASM_URL}
	URL_HASH ${NASM_HASH}

	#Setup Paths
	TMP_DIR ${TOOLCHAIN_TMP_DIR}
	STAMP_DIR ${TOOLCHAIN_STAMP_ROOT}/nasm-${TOOLCHAIN_BINUTILS_VERSION}-stamp
	DOWNLOAD_DIR ${TOOLCHAIN_DIST_DIR}
	SOURCE_DIR ${TOOLCHAIN_SRC_ROOT}/nasm-${TOOLCHAIN_BINUTILS_VERSION}
	BINARY_DIR ${TOOLCHAIN_BUILD_ROOT}/nasm-${TOOLCHAIN_NASM_VERSION}
	INSTALL_DIR ${TOOLCHAIN_PREFIX}

	CONFIGURE_COMMAND ${TOOLING_SRC_DIR}/nasm/configure --prefix=${TOOLING_PREFIX}
	BUILD_COMMAND make
)
