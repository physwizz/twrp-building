# twrp-building

TWRP Building For Beginners

1. Setup

sudo apt update
sudo apt upgrade
sudo apt-get install git-all

sudo apt install python-is-python3
sudo apt install python3-pip

    # for github.com 
git config --global user.email "your email"
git config --global user.name "your name"

2. swapfile

sudo swapoff -a

sudo dd if=/dev/zero of=/swapfile bs=1G count=8
sudo mkswap /swapfile
sudo swapon /swapfile

free -m

3. repo


mkdir -p ~/.bin
PATH="${HOME}/.bin:${PATH}"
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo

4. Basic Device Tree

4.1. setup twrpdtgen

mkdir ~/TWRP
cd ~/TWRP
git init
git clone https://github.com/twrpdtgen/twrpdtgen
pip3 install twrpdtgen
sudo apt install cpio

4.2. copy stock recovery.img to ~/TWRP/twrpdtgen

cd twrpdtgen
python3 -m twrpdtgen recovery.img

4.3. rename ~/TWRP/twrpdtgen/output/samsung/a12s/omni_a12s.mk to twrp_a12s.mk

4.4. delete vendorsetup.sh

4.5. to add extra partitions in recovery.fstab

    # V3
/system_image  emmc  /dev/block/mapper/system  flags=backup=0;flashimg=1;display="System Image"

/vendor_image  emmc  /dev/block/mapper/vendor  flags=backup=0;flashimg=1;display="Vendor Image"

/product_image  emmc  /dev/block/mapper/product  flags=backup=0\;flashimg=1\;display=\"Product Image\"";

/odm_image  emmc  /dev/block/mapper/odm  flags=backup=0;flashimg=1;display="Odm Image"

    # External 
/external_sd auto  /dev/block/mmcblk1p1 /dev/block/mmcblk1 flags=storage;wipeingui;removable

/usb-otg auto  /dev/block/sda1 /dev/block/sda   flags=display="USB-OTG";storage;wipeingui;removable

5.1
Change

    # /sdcard        sdfat     /dev/block/mmcblk1p1                                 flags=display=sdcard

    Also
    # /preload       ext4      /dev/block/platform/bootdevice/by-name/hidden        flags=display=preload

    # /keydata       ext4      /dev/block/platform/bootdevice/by-name/keydata       flags=display=keydata

    # /keyrefuge     f2fs      /dev/block/platform/bootdevice/by-name/keyrefuge     flags=display=keyrefuge

To add system wipe

/system             ext4    /dev/block/mapper/system                              flags=display=system;backup=1;wipeingui

6. Make a copy of recovery.fstab and rename to twrp.flags

7. Move both files recovery.fstab and twrp.flags to recovery/root/system/etc
(create these folders)

8. open twrp_a12s.mk and 
8.1. delete the lines below # Inherit from those products, Most specific first.
replace these lines with this
$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_base.mk)

8.2. delete $(call inherit-product, vendor/omni/config/gsm.mk)
8.3. change omni to twrp

9. open AndroidProducts.mk
change omni to twrp (4 spots)

10. Add to boardconfig.mk

    TW_NO_SCREEN_TIMEOUT := true
    TW_NO_SCREEN_BLANK := true
    #TW_SCREEN_BLANK_ON_BOOT := true
    TW_DEVICE_VERSION := 1_physwizz

To save space
-----------------------
You may have to compress kernel image or reduce the size of kernel
BOARD_KERNEL_IMAGE_NAME := Image.gz
TARGET_PREBUILT_KERNEL := $(DEVICE_PATH)/prebuilt/Image.gz

    # compress ramdisk
BOARD_RAMDISK_USE_LZMA := true
LZMA_RAMDISK_TARGETS := recovery

    # To save more space
BOARD_HAS_NO_REAL_SDCARD := true

To add fastbootd 

On BoardConfig.mk
TW_INCLUDE_FASTBOOTD := true

On device.mk add this

    # fastbootd
PRODUCT_PACKAGES += \
    android.hardware.fastboot@1.0-impl-mock \
    fastbootd

Using AIK to unpack and repack can also reduce size 

Establish Android Building Environment

cd ~
sudo apt install git aria2 -y
git clone https://gitlab.com/OrangeFox/misc/scripts
cd scripts
sudo bash setup/android_build_env.sh
sudo bash setup/install_android_sdk.sh

11. TWRP 11

mkdir ~/twrp-11

cd ~/twrp-11

repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-11

repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j4

* copy ~/TWRP/twrpdtgen/output/samsung 
to /twrp-11/device/samsung

12. Building

export ALLOW_MISSING_DEPENDENCIES=true
. build/envsetup.sh
lunch twrp_a12s-eng
mka recoveryimage -j4
