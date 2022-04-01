This will consist of all the unsuccessful and successful builds.

**Build 0**

Replace zimage and kernel_offset file
Replace ramdisk/etc/recovery.fstab file

Bootloader skipped verification and loaded the twrp from nand flash into ram. Twrp found appropriate kernel and kernel_offset correctly, loaded kernel and ramdisk, jumped back to kernel offset.

Kernel started execution, loaded the drivers, possibly missed or skipped touch drivers as mtk presumably disables touch in recovery. Kernel executed ramdisk/sbin/init which ran some scripts to enable other programs communicate with each other and to deliver the full twrp experience. Then the kernel transited control to the initial ramdisk at ramdisk_offset.

The ramdisk/sbin/init scripts are distinct from boot and stock recovery init scripts, started a small recovery c program in lieu of the java vm.


Searching for the text "tpd_i2c" in stock recovery kernel log file /cache/recovery/last_kmsg unveiled that the touch driver was indeed removed during initiation:

`
<4>[    1.098785] -(2)[53:kworker/u8:1][<c092e578>] (gt1x_deinit) from [<c092f0bc>] (tpd_i2c_remove+0x10/0x28)
<4>[    1.098796] -(2)[53:kworker/u8:1] r5:cf72d200 r4:c14a81cc
<4>[    1.098806] -(2)[53:kworker/u8:1][<c092f0ac>] (tpd_i2c_remove) from [<c0939660>] (i2c_device_remove+0x64/0x9c)
<4>[    1.098816] -(2)[53:kworker/u8:1][<c09395fc>] (i2c_device_remove) from [<c053d230>] 
`



**Build 1**

Minimal patch on build 1 and onwards:

*Unmodified in twrp split_img as these are the same in stock and twrp:*
base
kernel_offset
imgtype
header_version
hashtype
second_offset
ramdiskcomp
ramdisk_offset
pagesize tags_offset

*Unmodified in twrp split_img:*
Cmdline
origsize
recovery.img-ramdisk.cpio.gz

*Replacement in twrp split_img:*
board
os_version
os_patch_level
kernel (touch enabled)

*Addition to twrp split_img:*
avbtype
sigtype


*Unmodified in twrp ramdisk:*
default.prop symlink to prop.default
sbin/ueventd symlink to init

*Replacement in twrp ramdisk:*
prop.default
etc/recovery.fstab
ueventd.rc

*Addition to twrp ramdisk:*
fstab.enableswap

Replaced files were actually their respective stock version except the etc/recovery.fstab file in twrp ramdisk - which had the following additional lines at the end for external sdcard and usb otg support:

These lines may require manual formatting due to copying from a twrp fstab written in v1 with semicolon delimited flags:

`
/dev/block/mmcblk1* /external_sd auto flags=display="External SDcard";storage;wipeingui;removable;backup=0
/dev/block/sda* /usbotg auto flags=display="USB-OTG";storage;removable;backup=0
`

RESULT

kernel file size mismatch resulted in kernel offset anomaly and eventual futile recovery boot before normal boot.


**Build 2**
The touch enabled but size mismatched kernel was superseded with the stock kernel which had disabled touch.

RESULT

Recovery booted up in twrp successfully and no data decryption issue was encountered. But the touch sensor was disabled and locking twrp screen with power button was the only way of interaction with twrp.



**Build 3**
Fixed touch in stock kernel located in in `res/recovery_stock/split_img/recoverys.img_kernel` following numerous tutorials.
Check [no touch in stock kernel ISSUE](https://github.com/theintel/twrpwaltonprimoh8/issues/10) for discussion and learning resources on the issue.
Check this [readme entry in bugfix](~/bugfix/touchfixinkernel.md) for detailed fixation procedure.

RESULT
Supplantation of stock kernel by the new touch sensor hex fixed kernel and repacking before flashing the recovery successfully booted up in twrp with touch sensor enabled.
cache/recovery/last_kmsg last stock kernel log:

`
[    1.735551] -(0)[194:kworker/u8:3][<c0288c78>] (enable_irq) from [<c092f428>] (gt1x_irq_enable+0x38/0x50)
[    1.735606] -(0)[194:kworker/u8:3] r6:c1404948 r5:60000113 r4:c17f9ba0
[    1.735660] -(0)[194:kworker/u8:3][<c092f3f0>] (gt1x_irq_enable) from [<c092fb48>] (tpd_i2c_probe+0x1e0/0x2f4)
[    1.735714] -(0)[194:kworker/u8:3] r5:00000000 r4:c17f9ba0
[    1.735756] -(0)[194:kworker/u8:3][<c092f968>] (tpd_i2c_probe) from [<c0939870>] (i2c_device_probe+0x1d8/0x20c)
[    1.735811] -(0)[194:kworker/u8:3] r8:c092f968 r7:cf748a04 r6:cf748a20 r5:cf748a00
[    1.735877] -(0)[194:kworker/u8:3][<c0939698>] (i2c_device_probe) from [<c053cedc>] (driver_probe_device+0x318/0x470)
`

external sdcard and usb otg is parsed by voldlines in /ramdisk/etc/recovery.fstab and not by mmcblk1* and sda* due to unsupported wildcards in twrp 3.3.1 which is evident in recovery log:

*working usb otg partition by voldline:*
`
/auto2 |  | Size: 0MB Used: 0MB Free: 0MB Backup Size: 0MB
   Flags: Can_Be_Mounted Can_Be_Wiped Wipe_Available_in_GUI Removable Is_Storage 
   Display_Name: Storage
   Storage_Name: Storage
   Backup_Path: /auto2
   Backup_Name: auto2
   Backup_Display_Name: Storage
   Storage_Path: /auto2
   Current_File_System: vfat
   Fstab_File_System: vfat
   Backup_Method: files
   MTP_Storage_ID: 65540
`

 *Detection:*

`
I:Found a match 'sda' '/devices/platform/mt_usb'
I:Decrypt adopted storage starting
I:PageManager::LoadFileToBuffer loading filename: '/data/system/storage.xml' directly
I:successfully loaded storage.xml
I:No adopted storage so finding actual block device
I:Processing '/auto2-1'
I:Created '/auto2-1' folder.
/auto2-1 | /dev/block//sda1 | Size: 60484MB Used: 14485MB Free: 45998MB Backup Size: 14485MB
   Flags: Can_Be_Mounted Can_Be_Wiped Wipe_Available_in_GUI Is_SubPartition Removable IsPresent Is_Storage 
   SubPartition_Of: /auto2
   Primary_Block_Device: /dev/block//sda1
   Display_Name: Storage 1
   Storage_Name: Storage 1
   Backup_Path: /auto2-1
   Backup_Name: auto2-1
   Backup_Display_Name: auto2-1
   Storage_Path: /auto2-1
   Current_File_System: vfat
   Fstab_File_System: auto
   Backup_Method: files
`

"Select storage" tab in backup option shows it as "Storage 1"

Mixing semicolon delimited flags for fstab v1 with comma delimited flags for fstab v1 caused twrp not to parse flags in v1.

*not working usb otg partition by standard twrp line at the end:*

`
/usbotg |  | Size: 0MB
   Flags: Can_Be_Wiped 
   Primary_Block_Device: /dev/block/sda*
   Display_Name: usbotg
   Storage_Name: usbotg
   Backup_Path: /usbotg
   Backup_Name: usbotg
   Backup_Display_Name: usbotg
   Storage_Path: /usbotg
   Current_File_System: auto
   Fstab_File_System: auto
   Backup_Method: files
   Mount_Flags: 0, Mount_Options: flags=display="USB-OTG";storage;removable;backup=0
`



**Build 4**
edit lines in *ramdisk/etc/recovery.fstab*
comment out voldmanaged=flags appending # at the start:
`
# /devices/bootdevice* auto vfat defaults voldmanaged=sdcard0:auto
# /devices/platform/externdevice* auto auto defaults voldmanaged=sdcard1:auto,encryptable=userdata
# /devices/platform/mt_usb* auto vfat defaults voldmanaged=usbotg:auto
`

RESULT
No error and no auto0 auto1 auto2 storage creation in recovery.log but usb otg and external sdcard are not mounted anymore owing to commenting out voldlines and twrp not actually supporting wildcards in fstab v2.
Check ERROR3 in bug fix log for details.




**Build 5**
Transform fstab flags v1 to v2 by removing flags= and replace semicolons with commas.
Remove wildcards by writing the default partitions of external storage and usb otg:
/dev/block/mmcblk1p1 /external_sd auto display="External SDcard",storage,wipeingui,removable,backup=0
/dev/block/sda1 /usbotg auto display="USB-OTG",storage,removable,backup=0


RESULT
If no otg is connected, 2 warnings are logged during backup but none during backup deletion in restore tab. If an otg is connected, no warning is shown during backup and mount otg is tickable after 30s.
otg is unmountable when pwd is in a /usbotg* subdirectory.
Preconnected otg is mountable after 30s since russia splash screen shows up. So the kernel initiation time is around 30s.
reconnection after twrp boot and reopening mount tab makes otg mountable
3s for reconnect after complete twrp boot.
otg is not extant in select storage list.
otg to sdcard file copy and vice versa working but copying folder to otg takes long time.

Removing wildcards seem to cause the log to include sightly different lines. Mount options in twrp also included unelectable external storage and usb otg for the first 30 seconds. Every press prompted the log to include:
I:Unable to mount '/external_sd'
I:Actual block device: '', current file system: 'auto'
Failed to mount '/usbotg' (No such device)
I:Actual block device: '', current file system: 'auto'

But this time there was Size and Used data check in mount log:
/external_sd |  | Size: 0MB Used: 0MB Free: 0MB Backup Size: 0MB
   Flags: Can_Be_Mounted Can_Be_Wiped Wipe_Available_in_GUI Removable Is_Storage 
   Primary_Block_Device: /dev/block/mmcblk1p1
/usbotg |  | Size: 0MB Used: 0MB Free: 0MB Backup Size: 0MB
   Flags: Can_Be_Mounted Can_Be_Wiped 
   Primary_Block_Device: /dev/block/sda1

First detection and mount:
I:Found no matching fstab entry for uevent device '/devices/platform/mt_usb/musb-hdrc/usb1/1-1/1-1:1.0/host0/target0:0:0/0:0:0:0/block/sda' - add

Removal:
I:Found no matching fstab entry for uevent device '/devices/platform/mt_usb/musb-hdrc/usb1/1-1/1-1:1.0/host0/target0:0:0/0:0:0:0/block/sda' - remove
Failed to mount '/usbotg' (No such file or directory)
I:Actual block device: '/dev/block/sda1', current file system: 'vfat'

Different host2 mount in third connection:
I:Found no matching fstab entry for uevent device '/devices/platform/mt_usb/musb-hdrc/usb1/1-1/1-1:1.0/host2/target2:0:0/2:0:0:0/block/sda' - add
I:Set page: 'main'

EVEN host1 mount was noticed when connecting usb otg drive in normally booted android.

recovery.log indicates that the v2 flags were parsed as usual but usb otg storage detection was not working for the first 30s since twrp boot. But the kernel created /dev/block/sda block device files immediately with usb otg attachment which was manifest through twrp file manager.



**BUILD 6**
replace BUILD 3 recovery.fstab with stock recoverys.fstab

RESULT
The vold drives were actually processed successfully in build 3:
I:Unable to mount '/auto0'
I:Actual block device: '', current file system: 'vfat'

*UNABLE TO MOUNT indicates the storage may not be available while FAILED TO MOUNT indicates twrp presaged the device to be available.*

All the necessary flags were parsed automatically:
/auto0 |  | Size: 0MB Used: 0MB Free: 0MB Backup Size: 0MB
   Flags: Can_Be_Mounted Can_Be_Wiped Wipe_Available_in_GUI Removable Is_Storage 

But the Primary_Block_Device: /dev/block//sda1 was absent preliminary which was only included upon osb otg connection and mount:
:Processing '/auto2-1'
I:Created '/auto2-1' folder.
SubPartition_Of: /auto2
   Primary_Block_Device: /dev/block//sda1

The mount points were auto generated as /auto0 /auto1 /auto2 as per the instructions. The display names were all set to Storage automatically with mount points making them distinct.

usb otg detection in recovery.log:

I:Found a match 'sda' '/devices/platform/mt_usb'
I:Decrypt adopted storage starting
I:PageManager::LoadFileToBuffer loading filename: '/data/system/storage.xml' directly
I:successfully loaded storage.xml
I:No adopted storage so finding actual block device
I:Processing '/auto2-1'
I:Created '/auto2-1' folder.
/auto2-1 | /dev/block//sda1 | Size: 60484MB Used: 14461MB Free: 46022MB Backup Size: 14461MB
   Flags: Can_Be_Mounted Can_Be_Wiped Wipe_Available_in_GUI Is_SubPartition Removable IsPresent Is_Storage 
   SubPartition_Of: /auto2
   Primary_Block_Device: /dev/block//sda1
   Display_Name: Storage 1
   Storage_Name: Storage 1
   Backup_Path: /auto2-1
   Backup_Name: auto2-1
   Backup_Display_Name: auto2-1
   Storage_Path: /auto2-1
   Current_File_System: vfat
   Fstab_File_System: auto
   Backup_Method: files

Vold is noticeably slow in mounting and unmounting otg.

usb otg removal:
I:/auto2-1 was removed by uevent data
I:Set page: 'mount'
I:Is_Mounted: Unable to find partition for path '/auto2-1'
I:Is_Mounted: Unable to find partition for path '/auto2-1'