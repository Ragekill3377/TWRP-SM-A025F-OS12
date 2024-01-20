**How to Build Basic TWRP for a Android Device Android 9+​**
My XDA post:https://xdaforums.com/t/recovery-unnoficial-twrp-custom-recovery-for-sm-a025f-android-12.4651821/
​

###Note​###


***This is a basic method to build twrp. If twrp has bugs you have to fix it with your self according to your device
In samsung devices decryption/encryption cannot be fixed easily and it needs to modify kernel to fix mtp. most of time common bugs are mtp and decryption you need can find more info by referring more device trees**

#***DISCLAIMER***
 * Your warranty is now void.
* I am not responsible for bricked devices, dead SD cards,
* thermonuclear war, or your getting fired because the alarm app failed. Please
* do some research if you have any concerns about features included in this ROM
* before flashing it! YOU are choosing to make these modifications, and if
* you point the finger at me for messing up your device, I will laugh at you.
* If your device DOES get bricked, just flash old ROM, then patched vbmeta.img (you can compress to tar format with 7-zip easily), both in AP slot, then reflash twrp and to keep TWRP after reboot, turn off auto reboot in odin, use the key combination to exit to recovery from download, and flash the provided magisk.zip in twrp. after that, click "reboot" and reboot to system.
* If you still do not understand how to do this, or know basic commands or have knowledge about omni and android AOSP, best to leave this before you turn your phone into a useless brick.

#***FOR DEVELOPERS WHO WANT DEVICE TREE***#
I have made a device tree but at this time i am too lazy to get it. you can use the steps below to make the device tree and build twrp to your likings, or use the provided OFRP device tree for android 10 and 11 ONLY for SM-A025F!
D.T: ***<<<<https://gitlab.com/ForForkSake/a02s>>>>***
#WARNING: This link is a OFRP recovery device tree for SM-A025F OS 11! NOT OS 12! if you install this on OS 12, your phone will be softbricked and you will need to flash this new twrp which I've provided, or flash stock recovery.

#1.Prepare Environment

Create a github account​
Create a new empty repo​
Then log into gitpod using github account​
Open new workspace in gitpod,​
Select your new repo and selcect class as large​

​#***IF YOU HAVE LINUX/WSL YOU MAY SKIP THE ABOVE!(IF IT DOESNT WORK THEN FOLLOW ABOVE STEPS!)***#

#Make a device tree using twrpdtgen​

git clone https://github.com/twrpdtgen/twrpdtgen​
cd twrpdtgen​
sudo apt install cpio​
pip3 install twrpdtgen​
drag and drop the stock recovery.img to twrpdtgen folder​
python3 -m twrpdtgen <path to image>​
you will get twrp device tree at the out/manufature/code_name (eg: samsung/a02q)​
then copy manufature folder into root directory (workspace/name_of_your_github_repo)​
​
Install repo and packages​
sudo apt update​
sudo apt install rsync​
​
​
repo init --depth=1 --no-repo-verify -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1 -g default,-mips,-darwin,-notdefault​
repo sync -c --no-clone-bundle --no-tags --optimized-fetch --prune --force-sync -j8​
#Place device tree​

​

#move manufature folder (created device tree) to device (EG: workspace/name_of_your_github_repo/device/samsung)​

#Modify Device Tree​


#open device.mk and add these​

​
#Add support to fastbootd (skip the if your device dosnt have super.img)​

#Change the value according to shipped android os​

#if android 10 = 29​
#if android 11 = 30​
#if android 12 = 31​
#if android 12.1 = 32​
#if android 13 = 33​

PRODUCT_PACKAGES += \
    android.hardware.fastboot@1.0-impl-mock \
    fastbootd

PRODUCT_USE_DYNAMIC_PARTITIONS := true

PRODUCT_SHIPPING_API_LEVEL := 31
​
#Add device code name​

​
TARGET_OTA_ASSERT_DEVICE := a02q

TARGET_COPY_OUT_VENDOR := vendor



​
#TWRP FLAGS​

​
TARGET_RECOVERY_PIXEL_FORMAT := "RGBX_8888"
# TWRP specific build flags
TW_THEME := portrait_hdpi
RECOVERY_SDCARD_ON_DATA := true
TW_EXCLUDE_DEFAULT_USB_INIT := true
TW_EXTRA_LANGUAGES := true
TW_INCLUDE_NTFS_3G := true
TW_USE_TOOLBOX := true
TW_INCLUDE_RESETPROP := true
TW_INPUT_BLACKLIST := "hbtp_vm"
TW_BRIGHTNESS_PATH := "/sys/class/backlight/panel0-backlight/brightness"
TW_DEFAULT_BRIGHTNESS := 1200
TARGET_USES_MKE2FS := true
TW_NO_LEGACY_PROPS := true
TW_USE_NEW_MINADBD := true
TW_NO_BIND_SYSTEM := true
TW_NO_SCREEN_BLANK := true
TW_EXCLUDE_APEX := true
TW_FRAMERATE := 60
​
#TWRP Name​ for splash screen. you can add ANYTHING here!

​
TW_DEVICE_VERSION := Ragekill3377
​
#Enable notch​ (this is for notched devices. you can experiment around for desired value, but this is what i use)


TW_Y_OFFSET := 70
TW_H_OFFSET := -70
#Reboot to odin for samsung​

​
TW_HAS_DOWNLOAD_MODE := true​
​
#Enable Logcat​

​
TWRP_INCLUDE_LOGCAT := true[/HEADING]
[HEADING=2]TARGET_USES_LOGD := true​
​

#Modify fstab​


#Open recovery.fstab​

#add this to end of line to enable partition backup and flash​

;backup=1;flashimg

add those to enable sdcard and otg​

# Removable storage(since on samsung it still has a bug where dm verity cannot be disabled, so you must build custom kernel for that.
/external_sd    vfat        /dev/block/mmcblk1p1    /dev/block/mmcblk1        flags=storage;wipeingui;removable
/usb-otg    auto        /dev/block/sda1    /dev/block/sda                flags=display="USB-OTG";storage;wipeingui;removable


Open BoardConfig.mk and modify values according to your device (skip if you dont have super.img)​

# Dynamic Partition

BOARD_SUPER_PARTITION_SIZE := 3945791488 #TODO: Fix hardcoded value with actual size of super.img.lz4
BOARD_SUPER_PARTITION_GROUPS := android_dynamic_partitions
BOARD_ANDROID_DYNAMIC_PARTITIONS_SIZE := 3945791488 #TODO: Fox hardcoded calue with actual size of super.img.lz4
BOARD_ANDROID_DYNAMIC_PARTITIONS_PARTITION_LIST := system vendor product odm

Open Omni_a02q.mk​

$(call inherit-product, vendor/omni/config/common.mk)

rename omni to twrp​

$(call inherit-product, vendor/twrp/config/common.mk)

#Build TWRP​
**(If you face an error after build related to "expected expression" in 'graphics_fbdev.cpp", then replace the sprintf brightness line with ‘‘‘snprintf(brightness, 4, "%03d", TW_MAX_BRIGHTNESS/2);‘‘‘**

. build/envstup.sh
lunch (select your device-eng)
make recoveryimage

#***FULL YOUTUBE TUTORIAL AND STEPS BY SMILEY!***#
<<<<<<<https://youtu.be/ZYU5xJ2we1o?feature=shared>>>>>>>
