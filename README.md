

TWRP Building for Beginners

Note: I have used A127 as my example here

Taken from here
https://t.me/physwizz2/398

Part A
1. Setup


    sudo apt update
    sudo apt upgrade
    sudo apt-get install git-all

    sudo apt install python-is-python3
    sudo apt install python3-pip

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




Part B Basic Device Tree

1. setup twrpdtgen

    mkdir ~/TWRP

    cd ~/TWRP

    git init

    git clone https://github.com/twrpdtgen/twrpdtgen

    pip3 install twrpdtgen

    sudo apt install cpio


2. copy stock recovery.img to ~/TWRP/twrpdtgen

cd twrpdtgen


    python3 -m twrpdtgen recovery.img

3. rename ~/TWRP/twrpdtgen/output/samsung/a12s/omni_a12s.mk to twrp_a12s.mk

4. delete vendorsetup.sh

5. to add extra partitions in recovery.fstab

#added by physwizz

/system_image  emmc  /dev/block/mapper/system  flags=backup=0;flashimg=1;display="System Image"

/vendor_image  emmc  /dev/block/mapper/vendor  flags=backup=0;flashimg=1;display="Vendor Image"

/product_image  emmc  /dev/block/mapper/product  flags=backup=0\;flashimg=1\;display=\"Product Image\"";

/odm_image  emmc  /dev/block/mapper/odm  flags=backup=0;flashimg=1;display="Odm Image"

/external_sd	vfat		/dev/block/mmcblk1p1	/dev/block/mmcblk1	flags=storage;wipeingui;removable

/usb-otg	auto		/dev/block/sda1	/dev/block/sda			flags=display="USB-OTG";storage;wipeingui;removable

5.1
Change this

#/sdcard        sdfat     /dev/block/mmcblk1p1                                 flags=display=sdcard

5.2
You might to also want to do this:

#/preload       ext4      /dev/block/platform/bootdevice/by-name/hidden        flags=display=preload

#/keydata       ext4      /dev/block/platform/bootdevice/by-name/keydata       flags=display=keydata

#/keyrefuge     f2fs      /dev/block/platform/bootdevice/by-name/keyrefuge     flags=display=keyrefuge

6. Make a copy of recovery.fstab and rename to twrp.flags

7. Move both files recovery.fstab and twrp.flags to recovery/root/system/etc
(create these folders)

8. open twrp_a12s.mk and 
8.1. delete the 3 lines below # Inherit from those products, Most specific first.
replace those 3 lines with this

$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_base.mk)

8.2. delete $(call inherit-product, vendor/omni/config/gsm.mk)
8.3. change omni to twrp (2 places)

9. open AndroidProducts.mk
change omni to twrp (4 spots)

10. Add to boardconfig.mk


#added by physwizz

TW_NO_SCREEN_TIMEOUT := true
TW_NO_SCREEN_BLANK := true
#TW_SCREEN_BLANK_ON_BOOT := true
TW_DEVICE_VERSION := 1_physwizz


To save space

if not u have to compress kernel image or reduce the size of kernel
BOARD_KERNEL_IMAGE_NAME := Image.gz

#compress ramdisk
BOARD_RAMDISK_USE_LZMA := true
LZMA_RAMDISK_TARGETS := recovery

#To save more space
BOARD_HAS_NO_REAL_SDCARD := true

it will remove sd card partitioning and save some space

To add fastbootd

On device.mk add this

#fastbootd
PRODUCT_PACKAGES += \
    android.hardware.fastboot@1.0-impl-mock \
    fastbootd



Part C Establish Android Building Environment


cd ~
sudo apt install git aria2 -y
git clone https://gitlab.com/OrangeFox/misc/scripts
cd scripts
sudo bash setup/android_build_env.sh
sudo bash setup/install_android_sdk.sh


For TWRP 11 manifest

mkdir ~/twrp-11

cd ~/twrp-11

    repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-11

    repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j4


copy ~/TWRP/twrpdtgen/output/samsung 
to /twrp-11/device/samaung 

For TWRP 12 manifest


mkdir ~/twrp-12

cd ~/twrp-12

    repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1

    repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j4

    repo sync -j5 --current-branch --no-clone-bundle --no-tags

copy ~/TWRP/twrpdtgen/output/samsung 
to twrp-12/device/samsung

TW_THEME at
bootable/recovery/gui/libguitwrp_defaults.go

ui.xml at

TARGET_SUPPORTS_64_BIT_APPS := false


For TWRP 14.1 manifest


    repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_lineageos.git -b twrp-14.1

    repo sync -j5 --current-branch --no-clone-bundle --no-tags



    . build/envsetup.sh
    lunch lineageos_a24-eng
    mka recoveryimage -j4


For TWRP 10 manifest


    repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-10.0-deprecated


    repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j8


For TWRP 9 manifest



mkdir ~/twrp-9

cd ~/twrp-9


    repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-9.0

    repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j8


For TWRP 8 manifest


mkdir ~/twrp-8

cd ~/twrp-8

    repo init --depth=1 -u h://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-8.1

    repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j4


Part D To build for 8 & 9


sudo apt update
sudo apt install openjdk-17-jdk

sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk

sudo update-alternatives --config java
sudo update-alternatives --config javac


    export ALLOW_MISSING_DEPENDENCIES=true
    . build/envsetup.sh
    lunch omni_a01core-eng
    mka recoveryimage -j4




Part E Building 11 & 12


    export ALLOW_MISSING_DEPENDENCIES=true
    . build/envsetup.sh
    lunch twrp_a22x-eng
    mka recoveryimage -j4





Orange fox


mkdir ~/OrangeFox_sync
cd ~/OrangeFox_sync

git clone https://gitlab.com/OrangeFox/sync.git

cd ~/OrangeFox_sync/sync/

./orangefox_sync.sh --branch 11.0 --path ~/fox_11.0


* copy ~/TWRP/twrpdtgen/output/samsung to /fox_11.0/device/samsung


Building ofox


export ALLOW_MISSING_DEPENDENCIES=true
export FOX_USE_TWRP_RECOVERY_IMAGE_BUILDER=1
export LC_ALL="C"


. build/envsetup.sh
lunch twrp_a03s-eng && mka adbd recoveryimage -j4





