ARCH     = arm
XARCH    = $(ARCH)
CPU     ?= arm926t
MEM     ?= 128M
UBOOT   ?= v2015.07
# Need comment ORIDTB for 2.6.36 Linux
LINUX   ?= 4.6
KRN_ADDR?= 0x007fc0
RDK_ADDR?= 0x907fc0
DTB_ADDR?= 0x800000
UCONFIG ?= versatile.h
NETDEV  ?= smc91c111
SERIAL  ?= ttyAMA0
ROOTDEV ?= /dev/ram0
FSTYPE  ?= ext2
ORIIMG  ?= arch/$(ARCH)/boot/zImage
UORIIMG ?= arch/$(ARCH)/boot/uImage
ORIDTB  ?= arch/$(ARCH)/boot/dts/versatile-pb.dtb
CCPRE   ?= arm-linux-gnueabi-
BIMAGE  ?= $(PREBUILT_UBOOT)/$(XARCH)/$(MACH)/$(UBOOT)/u-boot
KIMAGE  ?= $(PREBUILT_KERNEL)/$(XARCH)/$(MACH)/$(LINUX)/zImage
DTB     ?= $(PREBUILT_KERNEL)/$(XARCH)/$(MACH)/$(LINUX)/versatile-pb.dtb
UKIMAGE ?= $(PREBUILT_KERNEL)/$(XARCH)/$(MACH)/$(LINUX)/uImage
ROOTFS  ?= $(PREBUILT_ROOT)/$(XARCH)/$(CPU)/rootfs.cpio.gz
UROOTFS ?= $(PREBUILT_ROOT)/$(XARCH)/$(CPU)/rootfs.cpio.uboot
HROOTFS ?= $(PREBUILT_ROOT)/$(XARCH)/$(CPU)/rootfs.$(FSTYPE)
