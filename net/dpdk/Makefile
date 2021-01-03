#
# Copyright (C) 2017 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=dpdk
PKG_VERSION=18.11
PKG_RELEASE=1

PKG_SOURCE_URL:=http://fast.dpdk.org/rel/
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_HASH:=76543b0a96f9e10a39a341eb137b363d26ece9b508d3f307668e91281d7a72fd

PKG_MANINTAINER:=Alexandru Ardelean <ardeleanalex@gmail.com>

PKG_BUILD_PARALLEL:=1
PKG_INSTALL:=1

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/dpdk/Default
  SECTION:=net
  CATEGORY:=Network
  URL:=http://dpdk.org
  DEPENDS:=@(i386||x86_64||arm||aarch64||powerpc64) +python +pciutils +@CONFIG_KERNEL_UIO
endef

define Package/dpdk
$(call Package/dpdk/Default)
  TITLE:=DPDK - Data Plane Development Kit
  DEPENDS+=
endef

TARGET_CFLAGS += \
	-D_GNU_SOURCE \
    -Wno-error=implicit-fallthrough= 

EXTRA_CFLAGS += \
    -mno-rdrnd \
    -mno-fsgsbase \
    -mavx \
    -mno-avx2 \
    -mno-avx512f

HOST_CFLAGS += \
	-D_GNU_SOURCE

DPDK_VARS:= \
	HOST_CC="$(HOSTCC)" \
	HOST_CPPFLAGS="$(HOST_CPPFLAGS)" \
	HOST_CFLAGS="$(HOST_CFLAGS)" \
	HOST_LDFLAGS="$(HOST_LDFLAGS)" \
    TARGET_CC="$(TARGET_CC)" \
	TARGET_CFLAGS="$(TARGET_CFLAGS)" \
	TARGET_LDFLAGS="$(TARGET_LDFLAGS)" \
	TARGET_ASFLAGS="$(TARGET_ASFLAGS)" \
	CROSS="$(TARGET_CROSS)" \
    RTE_KERNELDIR="$(LINUX_DIR)" \
    RTE_TARGET="$(ARCH)-native-linuxapp-gcc" \
    RTE_SDK="$(PKG_BUILD_DIR)" \
    RTE_OUTPUT="$(PKG_BUILD_DIR)/$(ARCH)-native-linuxapp-gcc" \
    EXTRA_CFLAGS="$(EXTRA_CFLAGS)" \
    kerneldir="/kernel"

CONFIGURE_VARS += \
	$(DPDK_VARS)

MAKE_VARS += \
	$(DPDK_VARS)

ifeq ($(ARCH),aarch64)
define Package/dpdk/config
ifeq ($(ARCH),arm64)
  source "$(SOURCE)/Config.in.arm64"
endef
endif

DPDK_ARCH:=$(ARCH)
# Do some ARCH adjustments
ifeq ($(ARCH),i386)
DPDK_ARCH:=i686
endif

# Adjust MACHINE
DPDK_MACHINE:=native
ifeq ($(ARCH),aarch64)
DPDK_ARCH:=arm64
DPDK_MACHINE:=$(CONFIG_DPDK_MACHINE)
endif
ifeq ($(ARCH),arm)
DPDK_MACHINE:=armv7a
endif

# These values are fixed
DPDK_EXECENV:=linuxapp
DPDK_TOOLCHAIN:=gcc

# The install rule will take care of everything
# There seems to be an issue with the build-system's `make` where
# it picks up `Makefile` first instead of `GNUmakefile` in
# the Build/Compile/Default rule.
# For now, we work around this
Build/Compile:=

MAKE_FLAGS += \
	T=$(DPDK_ARCH)-$(DPDK_MACHINE)-$(DPDK_EXECENV)-$(DPDK_TOOLCHAIN)

define Build/InstallDev
	$(INSTALL_DIR) $(1)/opt/dpdk
	$(CP) $(PKG_INSTALL_DIR)/* $(1)/opt/dpdk
endef

define Package/dpdk/install
	$(INSTALL_DIR) $(1)/bin $(1)/usr/share/dpdk $(1)/lib
	$(CP) $(PKG_INSTALL_DIR)/bin/* $(1)/bin
	$(CP) $(PKG_INSTALL_DIR)/share/dpdk/usertools $(1)/usr/share/dpdk/
	$(CP) $(PKG_INSTALL_DIR)/lib/* $(1)/lib
endef


define KernelPackage/dpdk-igb
  TITLE:=IGB DPDK Driver
  KCONFIG:= \
           CONFIG_HWPOISON_INJECT=y \
           CONFIG_HUGETLBFS=y \
           CONFIG_HUGETLB_PAGE=y \
           CONFIG_UIO=y \
           CONFIG_UIO_CIF \
           CONFIG_UIO_PDRV_GENIRQ \
           CONFIG_UIO_DMEM_GENIRQ \
           CONFIG_UIO_AEC \
           CONFIG_UIO_SERCOS3 \
           CONFIG_UIO_PCI_GENERIC \
           CONFIG_UIO_NETX \
           CONFIG_UIO_PRUSS \
           CONFIG_UIO_MF624 \
           CONFIG_UIO_HV_GENERIC
  FILES:=$(PKG_INSTALL_DIR)/kernel/igb_uio.ko
  AUTOLOAD:=$(call AutoLoad,30,igb_uio)
endef

define KernelPackage/loop/description
  Kernel module for loopback device support
endef

$(eval $(call KernelPackage,dpdk-igb))
$(eval $(call BuildPackage,dpdk))
