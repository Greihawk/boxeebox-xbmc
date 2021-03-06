cmake_minimum_required(VERSION 2.8)

set(TARGET "i686-pc-linux-gnu")
set(TARGET_DIR ${SYSROOT}/usr/local)
set(BASIC_CONF <SOURCE_DIR>/configure --prefix=${TARGET_DIR} --host=${TARGET} --enable-shared --disable-static --disable-nls)
set(BASIC_CMAKE CFLAGS=-I${TARGET_DIR}/include CC=${TARGET}-gcc CXX=${TARGET}-g++ RANLIB=${TARGET}-ranlib cmake -DPKG_CONFIG_EXECUTABLE=${TARGET}-pkg-config -DCMAKE_SYSTEM_NAME=Linux -DCMAKE_INSTALL_PREFIX=${TARGET_DIR} -DZLIB_ROOT=${SYSROOT}/usr -DZLIB_LIBRARIES=${SYSROOT}/usr/lib/libz.so -DZLIB_INCLUDE_DIRS=${SYSROOT}/usr/include <SOURCE_DIR>)

set(PYTHON_VERSION 2.7.2)

find_package(PythonInterp)

string(REGEX MATCH ${SYSROOT}/usr/bin SRPATH $ENV{PATH})

if(NOT SRPATH)
	message(FATAL_ERROR "${SYSROOT}/usr/bin must be in your path")
endif()

execute_process(COMMAND ${CMAKE_C_COMPILER} -dumpmachine OUTPUT_VARIABLE HOSTMACHINE)
string(REPLACE "\n" "" HOSTMACHINE ${HOSTMACHINE})

include(ExternalProject)

ExternalProject_Add(
	bzip2
	URL					http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz
	URL_MD5				00b516f4704d4a7cb50a1d97e6e8e15b
	PATCH_COMMAND		sed -e "s,CC=gcc,CC=${TARGET}-gcc," -e "s,RANLIB=ranlib,RANLIB=${TARGET}-ranlib," -e "s,PREFIX=/usr/local,PREFIX=${TARGET_DIR},g" -i Makefile
	CONFIGURE_COMMAND	""
	BUILD_COMMAND		""
	INSTALL_COMMAND		make install
	BUILD_IN_SOURCE 	1
)

ExternalProject_Add(
	lzo
	URL					http://www.oberhumer.com/opensource/lzo/download/lzo-2.06.tar.gz
	URL_MD5				95380bd4081f85ef08c5209f4107e9f8
	CONFIGURE_COMMAND	${BASIC_CONF}
)

ExternalProject_Add(
	zlib
	URL 				http://zlib.net/zlib-1.2.8.tar.gz
	URL_MD5 			44d667c142d7cda120332623eab69f40
	CONFIGURE_COMMAND	CROSS_PREFIX=${TARGET} AR=${TARGET}-ar CC=${TARGET}-gcc RANLIB=${TARGET}-ranlib <SOURCE_DIR>/configure --uname=Linux --prefix=${SYSROOT}/usr
	BUILD_COMMAND 		""
	INSTALL_COMMAND 	make ${PARALLEL} install
	BUILD_IN_SOURCE 	1
)

ExternalProject_Add(
	openssl
	URL 				ftp://ftp.mirrorservice.org/sites/ftp.openssl.org/source/openssl-1.0.1e.tar.gz
	URL_MD5 			66bf6f10f060d561929de96f9dfe5b8c
	CONFIGURE_COMMAND 	CC=${TARGET}-gcc RANLIB=${TARGET}-ranlib AR=${TARGET}-ar INSTALL_PREFIX=${TARGET_DIR} <SOURCE_DIR>/Configure linux-elf --openssldir=/
	DEPENDS zlib
	BUILD_IN_SOURCE 	1
)

ExternalProject_Add(
	libtiff
	URL 				http://download.osgeo.org/libtiff/tiff-4.0.3.tar.gz
	URL_MD5 			051c1068e6a0627f461948c365290410
	CONFIGURE_COMMAND	${BASIC_CONF}
)



ExternalProject_Add(
	flex
	URL 				http://sourceforge.net/projects/flex/files/flex-2.5.37.tar.bz2/download
	URL_MD5 			c75940e1fc25108f2a7b3ef42abdae06
	PATCH_COMMAND 		sed -e "s/AR\ =\ ar/AR\ =\ ${TARGET}-ar/g" -i Makefile.in
	CONFIGURE_COMMAND	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
)

ExternalProject_Add(
	expat
	URL 				http://sourceforge.net/projects/expat/files/expat/2.1.0/expat-2.1.0.tar.gz/download
	URL_MD5 			dd7dab7a5fea97d2a6a43f511449b7cd
	CONFIGURE_COMMAND	${BASIC_CONF}
)

ExternalProject_Add(
	yayl
	GIT_REPOSITORY  	https://github.com/lloyd/yajl.git
	GIT_TAG				2.0.1
	CONFIGURE_COMMAND 	${BASIC_CMAKE}
)

ExternalProject_Add(
	tinyxml
	URL 				http://sourceforge.net/projects/tinyxml/files/tinyxml/2.6.2/tinyxml_2_6_2.tar.gz/download
	URL_MD5 			c1b864c96804a10526540c664ade67f0
	CONFIGURE_COMMAND 	""
	BUILD_COMMAND 		make CXX=${TARGET}-g++ LD=${TARGET}-g++ CXXFLAGS=-DTIXML_USE_STL
		COMMAND ${TARGET}-ar r libtinyxml.a tinyxml.o tinystr.o tinyxmlerror.o tinyxmlparser.o
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy libtinyxml.a ${TARGET_DIR}/lib/
		COMMAND ${CMAKE_COMMAND} -E copy tinyxml.h ${TARGET_DIR}/include
		COMMAND ${CMAKE_COMMAND} -E copy tinystr.h ${TARGET_DIR}/include
	BUILD_IN_SOURCE 	1
)

ExternalProject_Add(
	sqlite
	URL 				http://www.sqlite.org/2013/sqlite-autoconf-3071700.tar.gz
	URL_MD5 			18c285053e9562b848209cb0ee16d4ab
	CONFIGURE_COMMAND	${BASIC_CONF}
)

ExternalProject_Add(
	libcdio
	URL 				http://ftp.gnu.org/gnu/libcdio/libcdio-0.90.tar.gz
	URL_MD5				1b245b023fb03a58d030fd2800db3247
	CONFIGURE_COMMAND 	${BASIC_CONF}
)

ExternalProject_Add(
	taglib
	URL 				http://taglib.github.io/releases/taglib-1.8.tar.gz
	URL_MD5 			dcb8bd1b756f2843e18b1fdf3aaeee15
	PATCH_COMMAND 		sed -e "54,61d" -i taglib/toolkit/taglib.h
	CONFIGURE_COMMAND	${BASIC_CMAKE}
)
ExternalProject_Add(
	libpng
	URL 				http://sourceforge.net/projects/libpng/files/libpng16/older-releases/1.6.3/libpng-1.6.3.tar.gz/download
	URL_MD5 			98B652B6C174C35148F1F5F513FE490D
	CONFIGURE_COMMAND 	CFLAGS=-I${TARGET_DIR}/include LDFLAGS=-L${TARGET_DIR}/lib ${BASIC_CONF}
	BUILD_COMMAND		CFLAGS=-I${TARGET_DIR}/include make ${PARALLEL}
	DEPENDS				zlib
)

ExternalProject_Add(
	pcre
	URL 				http://sourceforge.net/projects/pcre/files/pcre/8.33/pcre-8.33.tar.bz2/download
	URL_MD5 			21514018448ac10062ece7a668fca949
	CONFIGURE_COMMAND 	${BASIC_CONF}
)

ExternalProject_Add(
	jasper
	URL 				http://www.ece.uvic.ca/~frodo/jasper/software/jasper-1.900.1.zip
	URL_MD5 			a342b2b4495b3e1394e161eb5d85d754
	CONFIGURE_COMMAND 	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
)

ExternalProject_Add(
	libmicrohttpd
	URL 				ftp://ftp.gnu.org/gnu/libmicrohttpd/libmicrohttpd-0.9.28.tar.gz
	URL_MD5 			16c3cb66c839c6f929677471401a5b53
	PATCH_COMMAND 		sed -e "s,#ifdef LINUX,#ifdef LINUX\\n#define SOCK_NONBLOCK	O_NONBLOCK\\n#define EPOLL_CLOEXEC O_CLOEXEC," -e "s,epoll_create1,epoll_create,g" -i src/microhttpd/daemon.c
	CONFIGURE_COMMAND 	CFLAGS=-I{TARGET_DIR}/include LDFLAGS=-lc ${BASIC_CONF}
	DEPENDS 			openssl
)

ExternalProject_Add(
	freetype2
	URL 				http://sourceforge.net/projects/freetype/files/freetype2/2.5.0/freetype-2.5.0.1.tar.bz2/download
	URL_MD5 			c72e9010b1d986d556fc0b2b5fcbf31a
	CONFIGURE_COMMAND 	${BASIC_CONF} --without-png
	BUILD_COMMAND		make ${PARALLEL}
)

ExternalProject_Add(
	gdbm
	URL 				ftp://ftp.gnu.org/gnu/gdbm/gdbm-1.8.3.tar.gz
	URL_MD5 			1d1b1d5c0245b1c00aff92da751e9aa1
	PATCH_COMMAND		sed -e "s,finish_cmds=.*,,g" -i configure
		COMMAND 		sed -e "s,-o .(BINOWN) -g .(BINGRP),,g" -i Makefile.in
	CONFIGURE_COMMAND 	${BASIC_CONF}
)

ExternalProject_Add(
	fribidi
	URL 				http://fribidi.org/download/fribidi-0.19.5.tar.bz2
	URL_MD5 			925bafb97afee8a2fc2d0470c072a155
	CONFIGURE_COMMAND 	${BASIC_CONF}
	DEPENDS				freetype2
)

ExternalProject_Add(
	fontconfig
	URL 				http://www.freedesktop.org/software/fontconfig/release/fontconfig-2.10.93.tar.bz2
	URL_MD5 			0f8e5c63c2d6931626185c02802375a3
	CONFIGURE_COMMAND 	${BASIC_CONF}
	DEPENDS fribidi
)

ExternalProject_Add(
	libmad
	URL 				http://sourceforge.net/projects/mad/files/libmad/0.15.1b/libmad-0.15.1b.tar.gz/download
	URL_MD5 			1be543bc30c56fb6bea1d7bf6a64e66c
	PATCH_COMMAND 		sed -e "s/-fforce-mem//g" -i configure
	CONFIGURE_COMMAND 	${BASIC_CONF}
)

ExternalProject_Add(
	libjpeg-turbo
	URL 				http://sourceforge.net/projects/libjpeg-turbo/files/1.3.0/libjpeg-turbo-1.3.0.tar.gz/download
	URL_MD5 			e1e65cc711a1ade1322c06ad4a647741
    CONFIGURE_COMMAND   ${BASIC_CONF}
    BUILD_COMMAND       make
    INSTALL_COMMAND     make install
    BUILD_IN_SOURCE     1
)

ExternalProject_Add(
	fftw3
	URL 				http://www.fftw.org/fftw-3.3.3.tar.gz
	URL_MD5 			0a05ca9c7b3bfddc8278e7c40791a1c2
	CONFIGURE_COMMAND 	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
)

ExternalProject_Add(
	libsamplerate
	URL 				http://www.mega-nerd.com/SRC/libsamplerate-0.1.8.tar.gz
	URL_MD5 			1c7fb25191b4e6e3628d198a66a84f47
	CONFIGURE_COMMAND 	${BASIC_CONF}
)

ExternalProject_Add(
	libogg
	URL 				http://downloads.xiph.org/releases/ogg/libogg-1.3.1.tar.gz
	URL_MD5 			ba526cd8f4403a5d351a9efaa8608fbc
	CONFIGURE_COMMAND 	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
)


ExternalProject_Add(
	libvorbis
	URL 				http://downloads.xiph.org/releases/vorbis/libvorbis-1.3.3.tar.gz
	URL_MD5 			6b1a36f0d72332fae5130688e65efe1f
	CONFIGURE_COMMAND 	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
	DEPENDS libogg
)

ExternalProject_Add(
	flac
	URL 				http://downloads.xiph.org/releases/flac/flac-1.2.1.tar.gz
	URL_MD5 			153c8b15a54da428d1f0fadc756c22c7
	CONFIGURE_COMMAND 	${BASIC_CONF} --disable-oggtest --disable-xmms-plugin
	BUILD_COMMAND 		""
	INSTALL_COMMAND		make -C src ${PARALLEL} install
		COMMAND 		make -C include ${PARALLEL} install
	DEPENDS libvorbis
)

ExternalProject_Add(
	curl
	URL 				http://curl.haxx.se/download/curl-7.31.0.tar.bz2
	URL_MD5 			b7bea20579ac2f696338ae03f2c19ba5
	CONFIGURE_COMMAND 	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
	DEPENDS openssl zlib bzip2
)

ExternalProject_Add(
	libao
	URL 				http://downloads.xiph.org/releases/ao/libao-1.1.0.tar.gz
	URL_MD5 			2b2508c29bc97e4dc218fa162cf883c8
	CONFIGURE_COMMAND 	${BASIC_CONF}
	BUILD_COMMAND 		make ${PARALLEL}
)

ExternalProject_Add(
	vorbis-tools
	URL 				http://downloads.xiph.org/releases/vorbis/vorbis-tools-1.4.0.tar.gz
	URL_MD5 			567e0fb8d321b2cd7124f8208b8b90e6
	CONFIGURE_COMMAND 	LDFLAGS=-ldl ${BASIC_CONF}
	DEPENDS flac
)
ExternalProject_Add(
	libass
	URL 				https://libass.googlecode.com/files/libass-0.10.1.tar.gz
	URL_MD5 			6cace482a013a3c4bf3b31a68ac66026
	CONFIGURE_COMMAND 	${BASIC_CONF}
	DEPENDS libpng
)
# TODO:Fails due to libSDL
ExternalProject_Add(
	libmpeg2
	URL 				http://libmpeg2.sourceforge.net/files/libmpeg2-0.5.1.tar.gz
	URL_MD5 			0f92c7454e58379b4a5a378485bbd8ef
	CONFIGURE_COMMAND 	${BASIC_CONF} --disable-sdl
)

ExternalProject_Add(
	libmodplug
	URL 				http://sourceforge.net/projects/modplug-xmms/files/libmodplug/0.8.8.4/libmodplug-0.8.8.4.tar.gz/download
	URL_MD5 			fddc3c704c5489de2a3cf0fedfec59db
	CONFIGURE_COMMAND 	${BASIC_CONF}
)

ExternalProject_Add(
	libgpg-error
	URL 				ftp://ftp.gnupg.org/gcrypt/libgpg-error/libgpg-error-1.12.tar.bz2
	URL_MD5 			8f0eb41a344d19ac2aa9bd101dfb9ce6
	CONFIGURE_COMMAND 	${BASIC_CONF}
)
ExternalProject_Add(
	libgcrypt
	URL 				ftp://ftp.gnupg.org/gcrypt/libgcrypt/libgcrypt-1.5.3.tar.bz2
	URL_MD5 			993159b2924ae7b0e4eaff0743c2db35
	CONFIGURE_COMMAND 	GPG_ERROR_CONFIG=${TARGET_DIR}/bin/gpg-error-config ${BASIC_CONF} --disable-asm
	DEPENDS libgpg-error
)

ExternalProject_Add(
	libssh
	URL 				http://www.libssh.org/files/0.5/libssh-0.5.3.tar.gz
	URL_MD5 			9ad01838d3b89d98e900e0f6260a88cc
	PATCH_COMMAND 		sed -e "s,add_subdirectory\(examples\),,g" -i CMakeLists.txt
	CONFIGURE_COMMAND 	${BASIC_CMAKE} -DWITH_STATIC_LIB=ON -DCMAKE_SHARED_LINKER_FLAGS=-ldl -DCMAKE_EXE_LINKER_FLAGS=-ldl
	BUILD_COMMAND 		""
	INSTALL_COMMAND 	make install
	DEPENDS libgcrypt openssl
)

ExternalProject_Add(
	util-linux
	URL					https://www.kernel.org/pub/linux/utils/util-linux/v2.23/util-linux-2.23.2.tar.bz2
	URL_MD5				39a02ad0b3b37824c394f40930b7aa38
	CONFIGURE_COMMAND	${BASIC_CONF} --without-ncurses
	BUILD_COMMAND		""
	INSTALL_COMMAND		make install-uuidincHEADERS
)

ExternalProject_Add(
	samba
	URL 				http://ftp.samba.org/pub/samba/samba-3.6.20.tar.gz
	URL_MD5 			3f1b60c681845ce6828a1abe5aacf671
	CONFIGURE_COMMAND	CC=${TARGET}-gcc CFLAGS=-I${SYSROOT}/usr/local/include LDFLAGS=-L${SYSROOT}/usr/local/lib ${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/source3 ./configure --prefix=${TARGET_DIR} --host=${TARGET} samba_cv_CC_NEGATIVE_ENUM_VALUES=yes libreplace_cv_HAVE_GETADDRINFO=no ac_cv_file__proc_sys_kernel_core_pattern=yes  samba_cv_HAVE_STAT_ST_FLAGS=no samba_cv_DARWIN_INITGROUPS=no --without-ldap --without-krb5 --without-ads --disable-cups --enable-swat=no --with-winbind=no
	BUILD_COMMAND 		${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/source3 make ${PARALLEL} libsmbclient libtalloc libtevent libtdb
	INSTALL_COMMAND 	${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/source3 make ${PARALLEL} installlibsmbclient installlibtalloc installlibtevent installlibtdb
	BUILD_IN_SOURCE 	1
	DEPENDS 			util-linux
)

ExternalProject_Add(
        nfs
        URL               https://sites.google.com/site/libnfstarballs/li/libnfs-1.6.0.tar.gz
        URL_MD5           14a422a227745fa841f78a29f01492a4  
        PATCH_COMMAND     <SOURCE_DIR>/bootstrap --prefix=${TARGET_DIR} --host=${TARGET}
        CONFIGURE_COMMAND ${BASIC_CONF}
        BUILD_COMMAND     make
        INSTALL_COMMAND   make install
        BUILD_IN_SOURCE   1
)

ExternalProject_Add(
	rtmpdump
	GIT_REPOSITORY    git://git.ffmpeg.org/rtmpdump
	CONFIGURE_COMMAND ""
        BUILD_COMMAND     make DESTDIR=${SYSROOT} LIBS_posix=-L${TARGET_DIR}/lib XLIBS=-ldl INC=-I${TARGET_DIR}/include CROSS_COMPILE=${TARGET}-
        INSTALL_COMMAND   make install DESTDIR=${SYSROOT}
        BUILD_IN_SOURCE   1
        DEPENDS           zlib openssl
)

ExternalProject_Add(
        dbus_expat
        URL http://downloads.sourceforge.net/expat/expat-2.1.0.tar.gz
        URL_MD5 dd7dab7a5fea97d2a6a43f511449b7cd
        CONFIGURE_COMMAND ${BASIC_CONF} CC=${TARGET}-gcc
        BUILD_COMMAND make
        INSTALL_COMMAND make install
)

file(WRITE ${CMAKE_BINARY_DIR}/dbus_configure_helper.sh
"
export CC=${TARGET}-gcc
export CXX=${TARGET}-g++
export CFLAGS=-I${TARGET_DIR}/include
export LDFLAGS=-L${TARGET_DIR}/lib
export LIBS=\"-lpthread -lrt\"
${CMAKE_BINARY_DIR}/dbus-prefix/src/dbus/configure --prefix=${TARGET_DIR} --host=${TARGET} --without-x --disable-xml-docs --disable-doxygen-docs ac_cv_have_abstract_sockets=yes
")
execute_process(COMMAND chmod +x ${CMAKE_BINARY_DIR}/dbus_configure_helper.sh)

ExternalProject_Add(
        dbus
        URL               http://dbus.freedesktop.org/releases/dbus/dbus-1.6.14.tar.gz
        URL_MD5           1c22aebdf952e92908800f1ec2aeb0c4
        CONFIGURE_COMMAND ${CMAKE_BINARY_DIR}/dbus_configure_helper.sh
        BUILD_COMMAND     make
        INSTALL_COMMAND   make install
        BUILD_IN_SOURCE   1
        DEPENDS           dbus_expat
)

file(WRITE ${CMAKE_BINARY_DIR}/python_configure_helper.sh
"
export CC=${TARGET}-gcc
export CXX=${TARGET}-g++
export CPPFLAGS=\"-I${TARGET_DIR}/include -I${SYSROOT}/usr/include\"
export CFLAGS=-I${TARGET_DIR}/include
export LDFLAGS=\"-L${TARGET_DIR}/lib -L${SYSROOT}/usr/lib\"
export MACHDEP=Linux
export ac_sys_system=Linux
export ac_cv_prog_AR=${TARGET}-ar
export ac_cv_file__dev_ptmx=no
export ac_cv_file__dev_ptc=no
export enable_toolbox_glue=no
./configure --prefix=${TARGET_DIR} --host=${TARGET} --enable-shared --disable-ipv6  --build=${HOSTMACHINE} --target=${TARGET} --with-system-expat
")

execute_process(COMMAND chmod +x ${CMAKE_BINARY_DIR}/python_configure_helper.sh)

# Note to self: diff -x generated -r -u Python-2.7.2 python | grep -v '^Only in' > /Volumes/Skivavbild/boxee-xbmc/tools/boxeebox/libs/Python-2.7.2-xcompile.patch
ExternalProject_Add(
	python
	URL 				http://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tar.bz2
	URL_MD5 			ba7b2f11ffdbf195ee0d111b9455a5bd
	PATCH_COMMAND 	<SOURCE_DIR>/configure
		COMMAND make -j8 Parser/pgen python
		COMMAND mv python hostpython
		COMMAND mv Parser/pgen Parser/hostpgen
		COMMAND make distclean
		COMMAND patch -p1 -N < ${CMAKE_SOURCE_DIR}/Python-2.7.2-xcompile.patch
		COMMAND sed -e "13735,13738d" -i configure
		COMMAND sed -e "185d" -i Makefile.pre.in
		COMMAND sed -e "954,959d" -i Makefile.pre.in
	CONFIGURE_COMMAND   ${CMAKE_BINARY_DIR}/python_configure_helper.sh
		COMMAND rm -f Include/graminit.h
		COMMAND rm -f Python/graminit.c
		COMMAND Parser/hostpgen <SOURCE_DIR>/Grammar/Grammar Include/graminit.h Python/graminit.c
	BUILD_COMMAND 		HOSTPYTHON=./hostpython HOSTPGEN=./Parser/hostpgen CROSS_COMPILE_TARGET=yes HOSTARCH=${TARGET} BUILDARCH=${HOSTMACHINE} make ${PARALLEL}
	INSTALL_COMMAND 	HOSTPYTHON=./hostpython HOSTPGEN=./Parser/hostpgen CROSS_COMPILE_TARGET=yes HOSTARCH=${TARGET} BUILDARCH=${HOSTMACHINE} make install
	BUILD_IN_SOURCE 	1
	DEPENDS zlib expat bzip2 gdbm openssl sqlite
)

ExternalProject_Add(
	boost
	URL 				http://sourceforge.net/projects/boost/files/boost/1.54.0/boost_1_54_0.tar.bz2/download
	URL_MD5 			15cb8c0803064faef0c4ddf5bc5ca279
	PATCH_COMMAND 		${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/boost.user-config.jam ./tools/build/v2/user-config.jam
	CONFIGURE_COMMAND 	./bootstrap.sh --prefix=${TARGET_DIR}
	BUILD_COMMAND 		""
	INSTALL_COMMAND 	./b2 include=${TARGET_DIR}/include linkflags="-L${TARGET_DIR}/lib" toolset=gcc-i686 link=static target-os=linux --without-mpi --without-context --without-log --without-python --without-coroutine ${PARALLEL} install
	BUILD_IN_SOURCE 	1
	DEPENDS 			zlib bzip2
)

if(NOT EXISTS "${TARGET_DIR}/include/EGL/egl.h")
	message("Downloading egl headers")
	file(DOWNLOAD http://www.khronos.org/registry/egl/api/EGL/egl.h ${TARGET_DIR}/include/EGL/egl.h)
	file(DOWNLOAD http://www.khronos.org/registry/egl/api/EGL/eglext.h ${TARGET_DIR}/include/EGL/eglext.h)
	file(WRITE ${TARGET_DIR}/include/EGL/eglplatform.h "
#pragma once
#define EGLAPI
#define EGLAPIENTRY
typedef int NativeDisplayType;
typedef void *NativeWindowType;
typedef void *NativePixmapType;
typedef NativeDisplayType EGLNativeDisplayType;
typedef NativePixmapType  EGLNativePixmapType;
typedef NativeWindowType  EGLNativeWindowType;
typedef int EGLint;
")
	file(DOWNLOAD http://www.khronos.org/registry/egl/api/KHR/khrplatform.h ${TARGET_DIR}/include/KHR/khrplatform.h)
endif()

if(NOT EXISTS "${TARGET_DIR}/include/GLES2/gl2.h")
	message("Downloading gles2 headers")
	file(DOWNLOAD "http://www.khronos.org/registry/gles/api/GLES2/gl2.h" ${TARGET_DIR}/include/GLES2/gl2.h)
	file(DOWNLOAD "http://www.khronos.org/registry/gles/api/GLES2/gl2ext.h" ${TARGET_DIR}/include/GLES2/gl2ext.h)
	file(DOWNLOAD "http://www.khronos.org/registry/gles/api/GLES2/gl2platform.h" ${TARGET_DIR}/include/GLES2/gl2platform.h)
endif()

add_custom_command(OUTPUT ${TARGET_DIR}/lib/libEGL.so
	COMMAND cat ${TARGET_DIR}/include/EGL/egl.h | ${PYTHON_EXECUTABLE} -c \"import sys, re\; print '\#include <EGL/egl.h>\\n%s' % '\\n'.join(['%s {}' %x for x in re.compile(r'^\(EGLAPI\\s[^\;]+\)\;', re.M).findall(sys.stdin.read())])\" > ${CMAKE_BINARY_DIR}/libEGL.c
	COMMAND "${SYSROOT}/usr/bin/${TARGET}-gcc" ${CMAKE_C_FLAGS} -I${TARGET_DIR}/include -shared -o ${TARGET_DIR}/lib/libEGL.so ${CMAKE_BINARY_DIR}/libEGL.c
	DEPENDS "${TARGET_DIR}/include/EGL/egl.h"
)

add_custom_command(OUTPUT ${TARGET_DIR}/lib/libGLESv2.so
	COMMAND cat ${TARGET_DIR}/include/GLES2/gl2.h | ${PYTHON_EXECUTABLE} -c \"import sys, re\; print '\#include <GLES2/gl2.h>\\n%s' % '\\n'.join(['%s {}' %x for x in re.compile(r'^\(GL_APICALL\\s[^\;]+\)\;', re.M).findall(sys.stdin.read())])\" > ${CMAKE_BINARY_DIR}/libGLESv2.c
	COMMAND "${SYSROOT}/usr/bin/${TARGET}-gcc" ${CMAKE_C_FLAGS} -I${TARGET_DIR}/include -shared -o ${TARGET_DIR}/lib/libGLESv2.so ${CMAKE_BINARY_DIR}/libGLESv2.c
	DEPENDS "${TARGET_DIR}/include/GLES2/gl2.h"
)

add_custom_target(dummylibs ALL DEPENDS DEPENDS ${TARGET_DIR}/lib/libEGL.so ${TARGET_DIR}/lib/libGLESv2.so)

ExternalProject_Add(
	ffmpeg
	URL 				http://ffmpeg.org/releases/ffmpeg-1.2.2.tar.bz2
	URL_MD5 			4997660dfc077922e37d0583f9322f6b
	CONFIGURE_COMMAND 	<SOURCE_DIR>/configure --cross-prefix=${TARGET}- --enable-cross-compile --sysroot=${SYSROOT} --target-os=linux --arch=i686 --prefix=${TARGET_DIR} --enable-shared --disable-static --disable-programs --enable-gpl
	BUILD_COMMAND 		make ${PARALLEL}
)
