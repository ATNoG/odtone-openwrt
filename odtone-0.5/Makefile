#==============================================================================
# Brief   : ODTONE OpenWrt Makefile
# Authors : Carlos Guimarães <cguimaraes@av.it.pt>
#------------------------------------------------------------------------------
# ODTONE - Open Dot Twenty One
#
# Copyright (C) 2009-2013 Universidade Aveiro
# Copyright (C) 2009-2013 Instituto de Telecomunicações - Pólo Aveiro
#
# This software is distributed under a license. The full license
# agreement can be found in the file LICENSE in this distribution.
# This software may not be copied, modified, sold or distributed
# other than expressed in the named license agreement.
#
# This software is distributed without any warranty.
#==============================================================================

include $(TOPDIR)/rules.mk

PKG_NAME_SHORT:=odtone
PKG_NAME:=$(PKG_NAME_SHORT)
PKG_VERSION:=0.5

PKG_SOURCE:=$(PKG_NAME_SHORT)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:= http://atnog.av.it.pt/odtone/files/releases
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME_SHORT)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/odtone
  SECTION:=net
  CATEGORY:=Network
  URL:=http://atnog.av.it.pt/odtone
  DEPENDS:=+kmod-ipv6 +boost-thread +boost-system +boost-program_options +libnl
  TITLE:=ODTONE Package
endef

define Package/odtone/description
 This package contains the ODTONE implementation.
 ODTONE stands for Open Dot Twenty ONE and is an Open Source
 implementation of the Media Independent Handover framework
 from the IEEE 802.21 Media Independent Handover Services
 standard, using C++ APIs.
endef

define Package/Prepare
	$(call Build/Prepare/Default)
endef

define Build/Compile
	+( cd $(PKG_BUILD_DIR) ; \
		AR=$(GNU_TARGET_NAME)-ar \
		LD=$(GNU_TARGET_NAME)-ld \
		NM=$(GNU_TARGET_NAME)-nm \
		CC=$(GNU_TARGET_NAME)-gcc \
		GCC=$(GNU_TARGET_NAME)-gcc \
		CXX=$(GNU_TARGET_NAME)-g++ \
		RANLIB=$(GNU_TARGET_NAME)-ranlib \
		STRIP=$(GNU_TARGET_NAME)-strip \
		OBJCOPY=$(GNU_TARGET_NAME)-objcopy \
		OBJDUMP=$(GNU_TARGET_NAME)-objdump \
		b2 src/mihf app/sap_80211_linux \
			$(filter -j%,$(PKG_JOBS)) \
			toolset=gcc-$(ARCH) --build-type=minimal --layout=system optimization=space \
			--disable-long-double link=shared runtime-link=shared inlining=on debug-symbols=off \
			$(CONFIGURE_ARGS) linkflags="-lpthread -lgcc_eh -lboost_program_options -lboost_system -lboost_thread -lnl -lnl-genl -lnl-route" \
			release \
	)
endef

define Package/odtone/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/odtone $(1)/etc/init.d

	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_BUILD_DIR)/bin.v2/lib/external/libnl/gcc-mipsel/release/inlining-on/optimization-space/libnlwrap.so $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/bin.v2/lib/odtone/gcc-mipsel/release/inlining-on/optimization-space/libodtone.so $(1)/usr/lib/

	$(INSTALL_DIR) $(1)/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/dist/sap_80211 $(1)/sbin/

	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/dist/odtone-mihf $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/etc/odtone
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/dist/odtone.conf $(1)/etc/odtone/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/dist/sap_80211.conf $(1)/etc/odtone/
endef

$(eval $(call BuildPackage,odtone))
