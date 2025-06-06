#!/usr/bin/make -f
# Makefile for Cardinal #
# --------------------- #
# Created by falkTX
#

ROOT = ..
include $(ROOT)/Makefile.base.mk

# --------------------------------------------------------------
# override VCV arch.mk stuff so we can build more architectures

ifeq ($(CPU_ARM32),true)
ARCH_NAME = arm
MACHINE = arm64
else ifeq ($(CPU_ARM64),true)
ARCH_NAME = arm64
MACHINE = arm64
else ifeq ($(CPU_I386),true)
ARCH_NAME = i686
MACHINE = x86_64
else ifeq ($(CPU_RISCV64),true)
ARCH_NAME = riscv64
MACHINE = arm64
else ifeq ($(WASM),true)
ARCH_NAME = wasm
MACHINE = x86_64
else
MACHINE = $(TARGET_MACHINE)
endif

ifeq ($(MACOS),true)
MACHINE_SUFFIX = -darwin
else ifeq ($(WINDOWS),true)
MACHINE_SUFFIX = -mingw32
else
MACHINE_SUFFIX = -linux
endif

# --------------------------------------------------------------
# Set up env to pass to cmake and configure

ENV = env
ENV += AR=$(AR)
ENV += CC=$(CC)
ENV += CXX=$(CXX)
ENV += CFLAGS='$(BUILD_C_FLAGS)'
ENV += CXXFLAGS='$(BUILD_CXX_FLAGS)'
ENV += LDFLAGS='$(LINK_FLAGS)'

# --------------------------------------------------------------
# Fix up cmake

SPACE =
SPACE +=

CMAKE  = cmake
CMAKE += -DCMAKE_INSTALL_LIBDIR=lib
CMAKE += -DCMAKE_INSTALL_PREFIX='$(RACK_DEP_PATH)'
CMAKE += -DCMAKE_POLICY_VERSION_MINIMUM=3.5
CMAKE += -DBUILD_SHARED_LIBS=OFF

# make sure debug/release matches
ifeq ($(DEBUG),true)
CMAKE += -DCMAKE_BUILD_TYPE=Debug
else
CMAKE += -DCMAKE_BUILD_TYPE=Release
endif

# make sure macOS target matches ours
ifeq ($(MACOS),true)
ifneq (,$(findstring -arch$(SPACE),$(CXXFLAGS)))
CMAKE += -DCMAKE_OSX_ARCHITECTURES='$(subst $(SPACE),;,$(subst -arch=,,$(filter -arch=%,$(subst -arch$(SPACE),-arch=,$(CXXFLAGS)))))'
else ifeq ($(CIBUILD),true)
$(error CI build requires -arch flag on macOS)
endif
ifneq (,$(findstring -mmacosx-version-min=,$(CXXFLAGS)))
export MACOSX_DEPLOYMENT_TARGET = $(subst -mmacosx-version-min=,,$(filter -mmacosx-version-min=%,$(CXXFLAGS)))
CMAKE += -DCMAKE_OSX_DEPLOYMENT_TARGET=$(MACOSX_DEPLOYMENT_TARGET)
else ifeq ($(CIBUILD),true)
$(error CI build requires -mmacosx-version-min flag on macOS)
else
export MACOSX_DEPLOYMENT_TARGET = 10.15
CMAKE += -DCMAKE_OSX_DEPLOYMENT_TARGET=10.15
endif
CMAKE += -DCMAKE_OSX_SYSROOT="macosx"
endif

# fix emar and emranlib usage
ifeq ($(WASM),true)
CMAKE_AR     = $(shell which $(AR))
CMAKE_RANLIB = $(shell which $(RANLIB))
CMAKE += -DCMAKE_AR=$(CMAKE_AR)
CMAKE += -DCMAKE_C_COMPILER_AR=$(CMAKE_AR)
CMAKE += -DCMAKE_CXX_COMPILER_AR=$(CMAKE_AR)
CMAKE += -DCMAKE_RANLIB=$(CMAKE_RANLIB)
CMAKE += -DCMAKE_C_COMPILER_RANLIB=$(CMAKE_RANLIB)
CMAKE += -DCMAKE_CXX_COMPILER_RANLIB=$(CMAKE_RANLIB)
ifeq ($(shell uname -s),Darwin)
CMAKE += -DCMAKE_CROSSCOMPILING=ON
CMAKE += -DCMAKE_SYSTEM_NAME=Generic
endif
CMAKE += -DCMAKE_SKIP_COMPATIBILITY_TESTS=1
CMAKE += -DCMAKE_SIZEOF_CHAR=1
CMAKE += -DCMAKE_SIZEOF_UNSIGNED_SHORT=2
CMAKE += -DCMAKE_SIZEOF_SHORT=2
CMAKE += -DCMAKE_SIZEOF_INT=4
CMAKE += -DCMAKE_SIZEOF_UNSIGNED_LONG=4
CMAKE += -DCMAKE_SIZEOF_UNSIGNED_INT=4
CMAKE += -DCMAKE_SIZEOF_LONG=4
CMAKE += -DCMAKE_SIZEOF_VOID_P=4
CMAKE += -DCMAKE_SIZEOF_FLOAT=4
CMAKE += -DCMAKE_SIZEOF_DOUBLE=8
CMAKE += -DCMAKE_C_SIZEOF_DATA_PTR=4
CMAKE += -DCMAKE_CXX_SIZEOF_DATA_PTR=4
CMAKE += -DCMAKE_HAVE_LIMITS_H=1
CMAKE += -DCMAKE_HAVE_UNISTD_H=1
CMAKE += -DCMAKE_HAVE_PTHREAD_H=1
CMAKE += -DCMAKE_HAVE_SYS_PRCTL_H=1
CMAKE += -DCMAKE_WORDS_BIGENDIAN=0
CMAKE += -DCMAKE_DL_LIBS=''
CMAKE += -DCMAKE_C_BYTE_ORDER=LITTLE_ENDIAN
CMAKE += -DCMAKE_CXX_BYTE_ORDER=LITTLE_ENDIAN
endif

# fix cross-compilation for windows
ifeq ($(WINDOWS),true)
CMAKE += -G 'Unix Makefiles'
CMAKE += -DCMAKE_RC_COMPILER=$(subst gcc,windres,$(CC))
CMAKE += -DCMAKE_SYSTEM_NAME=Windows
endif

# --------------------------------------------------------------
# Fix up configure

CONFIGURE  = ./configure
CONFIGURE += --prefix="$(RACK_DEP_PATH)"
CONFIGURE += --host=$(TARGET_MACHINE)
CONFIGURE += --enable-static
CONFIGURE += --disable-shared
# NOTE libsamplerate wants to link against alsa, so we disable that
CONFIGURE += --disable-alsa
# NOTE speex fails to build when neon is enabled, so we disable that
CONFIGURE += --disable-neon
# NOTE libsamplerate fails with invalid host, so we force ac_cv_host
CONFIGURE += ac_cv_build=$(TARGET_MACHINE)
CONFIGURE += ac_cv_host=$(TARGET_MACHINE)

# --------------------------------------------------------------
# Fix up make

DEP_MAKE  = $(MAKE)
DEP_MAKE += ARCH_NAME=$(ARCH_NAME)
DEP_MAKE += AR=$(AR)
DEP_MAKE += CC=$(CC)
DEP_MAKE += CXX=$(CXX)
DEP_MAKE += CFLAGS="$(BUILD_C_FLAGS)"
DEP_MAKE += CXXFLAGS="$(BUILD_CXX_FLAGS)"
DEP_MAKE += LDFLAGS="$(LINK_FLAGS)"
DEP_MAKE += DEP_FLAGS="$(BASE_FLAGS)"
DEP_MAKE += DEP_MAC_SDK_FLAGS=
DEP_MAKE += MACHINE=$(MACHINE)$(MACHINE_SUFFIX)
DEP_MAKE += VERBOSE=1

ifeq ($(shell uname -s),Darwin)
ifeq ($(CIBUILD),true)
DEP_MAKE += SHA256SUM="shasum5.30 -a 256"
else
DEP_MAKE += SHA256SUM="shasum5.34 -a 256"
endif
endif

DEP_MAKE2  = $(DEP_MAKE)
DEP_MAKE2 += CMAKE="$(ENV) $(CMAKE)"
DEP_MAKE2 += CONFIGURE="$(ENV) $(CONFIGURE)"

# --------------------------------------------------------------
# Rack internal dependencies target

$(RACK_DEP_PATH)/lib/%.a:
	$(DEP_MAKE2) -C $(RACK_DEP_PATH) lib/$*.a

$(RACK_DEP_PATH)/jansson-2.12:
	$(DEP_MAKE2) -C $(RACK_DEP_PATH) jansson-2.12

# libarchive: skip shared lib and ensure libzstd is enabled
$(RACK_DEP_PATH)/lib/libarchive.a: $(RACK_DEP_PATH)/lib/libzstd.a $(RACK_DEP_PATH)/libarchive-3.4.3/.stamp-patched

$(RACK_DEP_PATH)/lib/libarchive_static.a: $(RACK_DEP_PATH)/lib/libzstd.a $(RACK_DEP_PATH)/libarchive-3.4.3/.stamp-patched

$(RACK_DEP_PATH)/libarchive-3.4.3/.stamp-patched:
	$(DEP_MAKE2) -C $(RACK_DEP_PATH) libarchive-3.4.3
	sed -i -e "618,625d" $(RACK_DEP_PATH)/libarchive-3.4.3/CMakeLists.txt
	awk 'NR==616{print "  SET(HAVE_LIBZSTD 1)"}1' $(RACK_DEP_PATH)/libarchive-3.4.3/CMakeLists.txt > $(RACK_DEP_PATH)/libarchive-3.4.3/CMakeLists.txt2
	mv $(RACK_DEP_PATH)/libarchive-3.4.3/CMakeLists.txt2 $(RACK_DEP_PATH)/libarchive-3.4.3/CMakeLists.txt
	sed -i -e "238,243d" $(RACK_DEP_PATH)/libarchive-3.4.3/libarchive/CMakeLists.txt
	sed -i -e "s/TARGETS archive archive_static/TARGETS archive_static/" $(RACK_DEP_PATH)/libarchive-3.4.3/libarchive/CMakeLists.txt
	touch $@

# libsamplerate: skip tests, fails to build in some systems and are not needed or wanted anyway
$(RACK_DEP_PATH)/lib/libsamplerate.a: $(RACK_DEP_PATH)/libsamplerate-0.1.9/.stamp-patched

$(RACK_DEP_PATH)/libsamplerate-0.1.9/.stamp-patched:
	$(DEP_MAKE2) -C $(RACK_DEP_PATH) libsamplerate-0.1.9
	sed -i -e "s/src doc examples tests/src/" $(RACK_DEP_PATH)/libsamplerate-0.1.9/Makefile.in
	touch $@

# libspeexdsp: hide symbols
$(RACK_DEP_PATH)/lib/libspeexdsp.a: $(RACK_DEP_PATH)/speexdsp/.stamp-patched

$(RACK_DEP_PATH)/speexdsp/.stamp-patched:
	sed -i -e 's/__attribute__((visibility("default")))//' $(RACK_DEP_PATH)/speexdsp/configure.ac
	touch $@

# custom zstd build for only building static libs
$(RACK_DEP_PATH)/lib/libzstd.a: $(RACK_DEP_PATH)/zstd-1.4.5/.stamp-patched
	cd $(RACK_DEP_PATH)/zstd-1.4.5/build/cmake && $(CMAKE) -DZSTD_BUILD_STATIC=ON -DZSTD_BUILD_PROGRAMS=OFF -DZSTD_BUILD_SHARED=OFF -DZSTD_MULTITHREAD_SUPPORT=OFF .
	$(DEP_MAKE2) -C $(RACK_DEP_PATH)/zstd-1.4.5/build/cmake
	$(DEP_MAKE2) -C $(RACK_DEP_PATH)/zstd-1.4.5/build/cmake install

# zstd cmake is borked, see https://github.com/facebook/zstd/issues/1401
# zstd also fails to build on old systems, patch that too
$(RACK_DEP_PATH)/zstd-1.4.5/.stamp-patched:
	$(DEP_MAKE2) -C $(RACK_DEP_PATH) zstd-1.4.5
	sed -i -e "56,66d" $(RACK_DEP_PATH)/zstd-1.4.5/build/cmake/CMakeModules/AddZstdCompilationFlags.cmake
	sed -i -e "146,175d" $(RACK_DEP_PATH)/zstd-1.4.5/programs/util.c
	sed -i -e "142,144d" $(RACK_DEP_PATH)/zstd-1.4.5/programs/util.c
	touch $@

# --------------------------------------------------------------
# QuickJS target, needed for AriaModules

QUICKJS_MAKE_FLAGS  = CFLAGS="$(BUILD_C_FLAGS) -D_GNU_SOURCE -DCONFIG_VERSION='\"Cardinal\"' -w"
QUICKJS_MAKE_FLAGS += PROGS=libquickjs.a

ifeq ($(WITH_LTO),true)
QUICKJS_MAKE_FLAGS += CONFIG_LTO=y
else
QUICKJS_MAKE_FLAGS += CONFIG_LTO=
endif

ifeq ($(WINDOWS),true)
QUICKJS_MAKE_FLAGS += CONFIG_WIN32=y
else ifeq ($(MACOS),true)
QUICKJS_MAKE_FLAGS += CONFIG_DARWIN=y
endif

$(RACK_DEP_PATH)/lib/libquickjs.a:
	$(DEP_MAKE) $(QUICKJS_MAKE_FLAGS) -C $(CURDIR)/QuickJS
	install -d $(RACK_DEP_PATH)/include
	install -d $(RACK_DEP_PATH)/lib
	install -m644 $(CURDIR)/QuickJS/libquickjs.a $@
	install -m644 $(CURDIR)/QuickJS/quickjs.h $(RACK_DEP_PATH)/include/quickjs.h

# --------------------------------------------------------------
# SurgeXT target

SURGE_DEP_PATH = $(abspath surge-build)
SURGE_SRC_PATH = $(abspath ../plugins/surgext/surge)
SURGE_NAMES = HysteresisProcessing Patch SolverType Tunings Wavetable clouds ghc plaits stmlib

SURGE_CXX_FLAGS  = $(filter-out -fsingle-precision-constant,$(filter-out -std=gnu++11,$(BUILD_CXX_FLAGS)))
SURGE_CXX_FLAGS += $(foreach n,$(SURGE_NAMES),-D$(n)=surgext$(n))

# fix build with gcc13
SURGE_CXX_FLAGS += -include cstdint

# fix JUCE build https://github.com/juce-framework/JUCE/issues/374
ifeq ($(CPU_I386),true)
SURGE_CXX_FLAGS += -D__sigemptyset=sigemptyset
endif

# use custom JUCE
SURGE_CXX_FLAGS += -I$(abspath ../deps/JUCE/modules)

# possibly use fftw?
# ifeq ($(shell $(PKG_CONFIG) --exists fftw3 fftw3f && echo true),true)
# SURGE_CXX_FLAGS += -DJUCE_DSP_USE_STATIC_FFTW=1
# endif

# JUCE_USE_CURL
SURGE_ENV = env \
	AR=$(AR) \
	CC=$(CC) \
	CXX=$(CXX) \
	CFLAGS='$(BUILD_C_FLAGS) -w' \
	CXXFLAGS='$(SURGE_CXX_FLAGS) -w' \
	LDFLAGS='$(LINK_FLAGS)'

SURGE_LIB = $(SURGE_DEP_PATH)/src/common/libsurge-common.a

$(SURGE_LIB): $(SURGE_DEP_PATH)/Makefile
	+$(DEP_MAKE) -C $(SURGE_DEP_PATH) surge-common

$(SURGE_DEP_PATH)/Makefile: $(SURGE_SRC_PATH)/CMakeLists.txt
	mkdir -p $(SURGE_DEP_PATH)
	cd $(SURGE_DEP_PATH) && \
		$(SURGE_ENV) $(CMAKE) \
		-DSURGE_BUILD_32BIT_LINUX=TRUE \
		-DSURGE_COMPILE_BLOCK_SIZE=8 \
		-DSURGE_SKIP_AIRWINDOWS=TRUE \
		-DSURGE_SKIP_JUCE_FOR_RACK=TRUE \
		-DSURGE_SKIP_LUA=TRUE \
		-DSURGE_SKIP_ODDSOUND_MTS=TRUE \
		-DSURGE_JUCE_PATH=$(abspath ../deps/JUCE) \
		-DSURGE_SIMDE_PATH=$(abspath ../src/Rack/dep/simde) \
		$(SURGE_SRC_PATH)

# --------------------------------------------------------------
# Build targets

ifneq ($(NOPLUGINS),true)
TARGETS += $(SURGE_LIB)
endif

TARGETS += $(RACK_DEP_PATH)/lib/libjansson.a
TARGETS += $(RACK_DEP_PATH)/lib/libquickjs.a
TARGETS += $(RACK_DEP_PATH)/lib/libsamplerate.a
TARGETS += $(RACK_DEP_PATH)/lib/libspeexdsp.a

ifeq ($(WINDOWS),true)
TARGETS += $(RACK_DEP_PATH)/lib/libarchive_static.a
else
TARGETS += $(RACK_DEP_PATH)/lib/libarchive.a
endif

TARGETS += $(RACK_DEP_PATH)/lib/libzstd.a

all: $(TARGETS)

clean:
	$(DEP_MAKE) $(QUICKJS_MAKE_FLAGS) -C $(CURDIR)/QuickJS clean
	rm -f $(TARGETS)
	rm -f $(RACK_DEP_PATH)/*.tgz
	rm -f $(RACK_DEP_PATH)/*.tar.gz
	rm -rf $(RACK_DEP_PATH)/bin
	rm -rf $(RACK_DEP_PATH)/include
	rm -rf $(RACK_DEP_PATH)/lib
	rm -rf $(RACK_DEP_PATH)/share
	rm -rf $(RACK_DEP_PATH)/jansson-2.12
	rm -rf $(RACK_DEP_PATH)/libarchive-3.4.3
	rm -rf $(RACK_DEP_PATH)/libsamplerate-0.1.9
	rm -rf $(RACK_DEP_PATH)/speexdsp/autom4te.cache
	rm -rf $(RACK_DEP_PATH)/speexdsp/libspeexdsp/.deps
	rm -rf $(RACK_DEP_PATH)/speexdsp/libspeexdsp/.libs
	rm -rf $(RACK_DEP_PATH)/speexdsp/libspeexdsp/*.lo
	rm -rf $(RACK_DEP_PATH)/speexdsp/libspeexdsp/*.o
	rm -rf $(RACK_DEP_PATH)/speexdsp/Makefile
	rm -rf $(RACK_DEP_PATH)/zstd-1.4.5
	rm -rf $(SURGE_DEP_PATH)

download: \
	$(RACK_DEP_PATH)/jansson-2.12 \
	$(RACK_DEP_PATH)/libarchive-3.4.3/.stamp-patched \
	$(RACK_DEP_PATH)/libsamplerate-0.1.9/.stamp-patched \
	$(RACK_DEP_PATH)/zstd-1.4.5/.stamp-patched

quickjs: $(RACK_DEP_PATH)/lib/libquickjs.a
surge: $(SURGE_LIB)

# --------------------------------------------------------------
