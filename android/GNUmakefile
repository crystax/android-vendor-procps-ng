override MYDIR := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))
override TOPDIR := $(abspath $(MYDIR)/..)

sinclude $(TOPDIR)/local.mk

#ABI ?= armeabi-v7a-hard
ABI ?= arm64-v8a

ifeq (,$(strip $(NDK)))
NDK := $(abspath $(wildcard $(TOPDIR)/../../platform/ndk))
endif

#==================================================================================================

define rm-if-exists
$(if $(wildcard $(1)),rm -Rf $(wildcard $(1)))
endef

define rmdir-if-exists
$(if $(wildcard $(1)),rmdir $(wildcard $(1)) >/dev/null 2>&1 || true)
endef

define out-dir
$(or $(strip $(OUTDIR)),$(TOPDIR)/android/out/$(1))
endef

define build-dir
$(call out-dir,$(1))/build
endef

define install-dir
$(call out-dir,$(1))/install
endef

#==================================================================================================

.PHONY: all
all: build

.PHONY: clean
clean:
	$(call rm-if-exists,$(call out-dir,$(ABI)))
	@$(call rmdir-if-exists,$(dir $(call out-dir,$(ABI))))

.PHONY: rebuild
rebuild:
	$(MAKE) clean
	$(MAKE) all

.PHONY: build
build:
	$(if $(strip $(NDK)),,$(error NDK was not defined!))
	$(if $(strip $(ABI)),,$(error ABI was not defined!))
	$(if $(strip $(NCURSES)),,$(error NCURSES was not defined!))
	@$(strip \
		export NDK=$(NDK) && \
		export ABI=$(ABI) && \
		export OUTDIR=$(call out-dir,$(ABI)) && \
		source $(TOPDIR)/android/envsetup.sh && \
		mkdir -p $(call build-dir,$(ABI)) && \
		cd $(call build-dir,$(ABI)) && \
		ac_cv_func_malloc_0_nonnull=yes \
		ac_cv_func_realloc_0_nonnull=yes \
		LIBS='-ltinfow' \
		$(TOPDIR)/configure \
			--prefix=$(call install-dir,$(ABI)) \
			--host=$$TARGET \
			--with-pic \
			--enable-static \
			--disable-shared \
			--enable-watch8bit \
			--disable-nls \
			--disable-rpath \
			&& \
		make -j1 && \
		make install && \
		true \
	)
