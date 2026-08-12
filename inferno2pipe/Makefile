# SPDX-License-Identifier: GPL-2.0-only

include $(TOPDIR)/rules.mk

PKG_NAME:=inferno2pipe
PKG_VERSION:=0.5.4
PKG_RELEASE:=1

PKG_LICENSE:=GPL-3.0-or-later OR AGPL-3.0-or-later
PKG_LICENSE_FILES:=LICENSE inferno_aoip/LICENSE-GPL inferno_aoip/LICENSE-AGPL

PKG_BUILD_DEPENDS:=rust/host
PKG_BUILD_PARALLEL:=1
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

MAKE_PATH:=inferno2pipe
RUST_PKG_LOCKED:=1

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/rust/rust-package.mk

define Package/inferno2pipe
  SECTION:=sound
  CATEGORY:=Sound
  TITLE:=Unofficial Dante-compatible receiver to a raw PCM pipe
  URL:=https://github.com/teodly/inferno
  DEPENDS:=$(RUST_ARCH_DEPENDS)
endef

define Package/inferno2pipe/description
 Inferno2pipe receives audio from Dante-compatible devices and writes
 interleaved signed 32-bit PCM samples to a file or Unix named pipe.
 Inferno is unofficial, experimental and not endorsed by Audinate.
endef

define Build/Prepare
	$(RM) -r $(PKG_BUILD_DIR)
	$(INSTALL_DIR) $(PKG_BUILD_DIR)
	$(CP) $(CURDIR)/src/. $(PKG_BUILD_DIR)/
endef

$(eval $(call RustBinPackage,inferno2pipe))
$(eval $(call BuildPackage,inferno2pipe))

