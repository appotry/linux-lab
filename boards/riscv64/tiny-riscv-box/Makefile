# Copyright (C) 2023-2024 Wu Zhangjin <falcon@tinylab.org>, All Rights Reserved.
# _BASE: 1, virt; 2, real; 3: virt+real
_BASE   := 2
ARCH    := riscv
XARCH   := riscv64
CPU     ?= cv1800b
SMP     ?= 1
MEM     ?= 64MB
LINUX   ?= v5.10.4

UBOOT   ?= v2021.10

BUILDROOT?= 2021.05

SERIAL  ?= ttyS0

ROOTDEV_LIST := /dev/mmcblk0
ROOTDEV ?= /dev/mmcblk0
FSTYPE  ?= ext4

ORIIMG  ?= arch/$(ARCH)/boot/Image.lzma
UORIIMG ?= $(ORIIMG)
KIMAGE  ?= $(BSP_KERNEL)/$(LINUX)/Image.lzma
UKIMAGE ?= $(KIMAGE)
KRELEASE?= $(BSP_KERNEL)/$(LINUX)/kernel.release

ORIDTB  ?= arch/$(ARCH)/boot/dts/cvitek/cv1800b_milkv_duo_sd.dtb
DTB     ?= $(BSP_KERNEL)/$(LINUX)/cv1800b_milkv_duo_sd.dtb

CCPATH	?= $(BSP_TOOLCHAIN)/riscv64-linux-musl-x86_64/bin
CCPRE	?= riscv64-unknown-linux-musl-
# Path to toolchain package
CCURL     ?= $(BSP_TOOLCHAIN)/toolchain.tar.xz
# Path to decompressing directory
TOOLCHAIN ?= $(BSP_TOOLCHAIN)

BIMAGE  ?= $(BSP_UBOOT)/$(UBOOT)/u-boot.bin

BIOS    ?= $(BSP_DIR)/opensbi/bin/fw_dynamic.bin

ROOTFS  ?= $(BSP_ROOT)/$(BUILDROOT)/rootfs.cpio.gz

# Prefer serial port login instead of ssh
COM ?= serial

# Caching build
CACHE_BUILD ?= 1

# Please list all available ip here, the first ping-able one will be used
BOARD_IP_LIST ?= 192.168.42.1

# OpenSBI build and save support, based on tiny-riscv-box branch
OPENSBI_BUILD = $(TOP_BUILD)/$(XARCH)/$(MACH)/opensbi
OPENSBI_BIOS := $(OPENSBI_BUILD)/platform/generic/firmware/fw_dynamic.bin

opensbi: opensbi-build
opensbi-build: $(OPENSBI_BIOS)
$(OPENSBI_BIOS): toolchain-install $(BSP_DIR)/opensbi/src
	$(Q)echo "LOG: Building OpenSBI ..."
	$(Q)env PATH=$(if $(CCPATH),$(CCPATH):)$(PATH) \
	make $(NPD) -C $(BSP_DIR)/opensbi/src \
	PLATFORM=generic FW_PAYLOAD_PATH=$(BIMAGE) FW_FDT_PATH=$(DTB) CROSS_COMPILE=$(CCPRE) \
	O=$(OPENSBI_BUILD)

opensbi-clean: $(BSP_DIR)/opensbi/src
	$(Q)make $(NPD) -C $(BSP_DIR)/opensbi/src O=$(OPENSBI_BUILD) clean

opensbi-save: $(OPENSBI_BIOS)
	$(Q)cp -v $^ $(BSP_DIR)/opensbi/bin/

# Use latest build opensbi instead of the old one
ifneq ($(wildcard $(OPENSBI_BIOS)),)
  BIOS := $(wildcard $(OPENSBI_BIOS))
endif

PHONY += opensbi $(addprefix opensbi-,build clean save)

# Upload support, require run /mnt/system/usb-rndis.sh after connect the main board to host
LOCAL_MODULES  ?= $(ROOTDIR)/lib/modules/
REMOTE_MODULES ?= /lib/modules/
# We don't have a rsync in the target system? use scp instead
RSYNC_CMD ?= $(SCP_CMD) -r

# Both kimage and dtb are put in boot.sd
IMAGE_BUILD = $(TOP_BUILD)/$(XARCH)/$(MACH)/image

LOCAL_KIMAGE ?= $(IMAGE_BUILD)/boot.sd
REMOTE_KIMAGE ?= /boot/boot.sd
LOCAL_DTB    ?= $(LOCAL_KIMAGE)
REMOTE_DTB   ?= $(REMOTE_KIMAGE)
LOCAL_BIMAGE ?= $(IMAGE_BUILD)/fip.bin
REMOTE_BIMAGE ?= /boot/fip.bin

# Export some file variables, make sure package the latest images to SD Card Image
export KIMAGE DTB BIOS BIMAGE ROOTFS

# Prepare local fip.bin and remote fip.bin store directory
$(LOCAL_BIMAGE): fip-bin remote-boot

fip-bin-deps: $(BIOS) $(BIMAGE) $(wildcard $(BSP_DIR)/firmwares/*)
fip.bin: fip-bin
fip-bin: fip-bin-deps
	$(Q)echo "LOG: Building $(LOCAL_KIMAGE)"
	$(Q)mkdir -p $(IMAGE_BUILD); cd $(IMAGE_BUILD); $(BSP_DIR)/tools/fip-bin.sh

# Prepare local boot.sd and remote boot.sd store directory
$(LOCAL_KIMAGE): boot-sd remote-boot

boot-sd-deps: $(DTB) $(KIMAGE) $(BSP_DIR)/configs/multi.its
boot.sd: boot-sd
boot-sd: boot-sd-deps
	$(Q)echo "LOG: Building $(LOCAL_KIMAGE)"
	$(Q)mkdir -p $(IMAGE_BUILD); cd $(IMAGE_BUILD); $(BSP_DIR)/tools/boot-sd.sh

remote-boot: FORCE
	$(Q)echo "LOG: Mounting remote SD Card boot partition from /dev/mmcblk0p1 to /boot/"
	$(Q)$(SSH_CMD) "mkdir -p /boot; grep -q '/dev/mmcblk0p1 /boot ' /proc/mounts || mount /dev/mmcblk0p1 /boot/" || true

PHONY += fip.bin boot.sd fip-bin boot-sd remote-boot fip-bin-deps boot-sd-deps

# Put all images together to one SD Card image, you must build and save the sub images manually
IMAGE_BIN = $(IMAGE_BUILD)/tiny-riscv-box.img
IMAGE_MODULES ?= hello,8723du

prepare-rootfs: root-dir
	$(Q)make root-dir-install-system
	$(Q)make root-rd

prepare-modules: root-dir
	$(Q)make modules-install
	$(Q)make modules-install m=$(IMAGE_MODULES)
	$(Q)make root-rd

prepare-opensbi: opensbi-save
prepare-uboot: uboot-save
prepare-kernel: kernel-save
prepare-dtb: prepare-kernel

prepare-image: $(addprefix prepare-,opensbi uboot kernel rootfs modules)

clean-image:
	$(Q)echo "LOG: Removing $(IMAGE_BIN) ..."
	$(Q)rm -rf $(IMAGE_BIN)

post-image: $(IMAGE_BIN)

ifeq ($(CACHE_BUILD),1)
  CACHE_BUILD_TARGET := cache-build
endif

$(call _stamp,image,outdir): $(CACHE_BUILD_TARGET)
	$(Q)mkdir -p $(IMAGE_BUILD)
	$(Q)touch $@

$(IMAGE_BIN): $(call _stamp,image,outdir) fip-bin-deps boot-sd-deps $(BSP_DIR)/configs/genimage.cfg FORCE
	$(Q)echo "LOG: Building $(IMAGE_BIN) ..."
	$(Q)cd $(IMAGE_BUILD); $(BSP_DIR)/tools/post-image.sh

burn-image:
	$(Q)echo "LOG: Burning $(IMAGE_BIN) ..."
	$(Q)if [ -z "$(SD_SIZE)" -o -z "$(SD_CARD)" ]; then \
	  echo "ERR: Please configure SD_CARD and SD_SIZE in $(BOARD_LABCONFIG) with these commands:"; \
	  echo ""; \
	  echo "     $$ make local-config SD_CARD=/path/to/your-sd-card"; \
	  echo "     $$ lsblk -dbno size /path/to/your-sd-card"; \
	  echo "     $$ make local-config SD_SIZE=the-above-bytes-of-your-sd-card"; \
	  echo ""; \
	  false; \
	fi
	$(Q)SD_IMAGE=$(IMAGE_BIN) SD_SIZE=$(SD_SIZE) SD_CARD=$(SD_CARD) SD_MAX=$(SD_MAX) $(BSP_DIR)/tools/burn-image.sh

# FIXME: Still not include the slow rootfs building, please run it manually with 'make root'
allinone: prepare-image post-image burn-image

PHONY += allinone post-image burn-image clean-image $(addprefix prepare-,opensbi uboot kernel rootfs modules image)
