This will consist of all the unsuccessful and successful builds.

**Build 0**

Replace zimage and kernel_offset file
Replace ramdisk/etc/recovery.fstab file

Bootloader skipped verification and loaded the twrp from nand flash into ram. Twrp found appropriate kernel and kernel_offset correctly, loaded kernel and ramdisk, jumped back to kernel offset.

Kernel started execution, loaded the drivers, possibly missed or skipped touch drivers as mtk presumably disables touch in recovery. Kernel executed ramdisk/sbin/init which ran some scripts to enable other programs communicate with each other and to deliver the full twrp experience. Then the kernel transited control to the initial ramdisk at ramdisk_offset.

The ramdisk/sbin/init scripts are distinct from boot and stock recovery init scripts, started a small recovery c program in lieu of the java vm.


Searching for the text "tpd_i2c" in stock recovery kernel log file /cache/recovery/last_kmsg unveiled that the touch driver was indeed removed during initiation:

<4>[    1.098785] -(2)[53:kworker/u8:1][<c092e578>] (gt1x_deinit) from [<c092f0bc>] (tpd_i2c_remove+0x10/0x28)
<4>[    1.098796] -(2)[53:kworker/u8:1] r5:cf72d200 r4:c14a81cc
<4>[    1.098806] -(2)[53:kworker/u8:1][<c092f0ac>] (tpd_i2c_remove) from [<c0939660>] (i2c_device_remove+0x64/0x9c)
<4>[    1.098816] -(2)[53:kworker/u8:1][<c09395fc>] (i2c_device_remove) from [<c053d230>] 



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


RESULT

kernel file size mismatch resulted in kernel offset anomaly and eventual futile recovery boot before normal boot.



**Build 2**
The touch enabled but size mismatched kernel was superseded with the stock kernel which had disabled touch.

RESULT

Recovery booted up in twrp successfully and no data decryption issue was encountered. But the touch sensor was disabled and locking twrp screen with power button was the only way of interaction with twrp.



**Build 3**
Installed magisk module "cross compiled binaries" by zackptg5 and installed original gzip v1.10 via terminal command "ccbins". After two reboots, command "gzip -h" demonstrated the flag "-n or --no--name". Recompression of the recovery.img-zImage_hexfix kernel file with command "gzip -n -k -9" resulted in compressed gzip kernel of size 6993326 which is 1 byte less than size of hexnotfix_errorfree gzip kernel. This nuanced size mismatch was fixed by appending 10 lines of eight "00" bytes or 80 bytes at the end of recovery.img-zImage_hexfix kernel which increased the file size from 22040576 to 22040656. This time the recompression created a gzip size of 6993327 bytes matching exactly with hexnotfix_errorfree gzip kernel.
After appending header and footer data the final recovery.img-zImage kernel file reached a size of 7077184 that matched with the stock kernel which had disabled touch.

RESULT
Supplantation of stock kernel by the new touch sensor hex fixed kernel and repacking before flashing the recovery successfully booted up in twrp with touch sensor enabled.
cache/recovery/last_kmsg last stock kernel log:

[    1.735551] -(0)[194:kworker/u8:3][<c0288c78>] (enable_irq) from [<c092f428>] (gt1x_irq_enable+0x38/0x50)
[    1.735606] -(0)[194:kworker/u8:3] r6:c1404948 r5:60000113 r4:c17f9ba0
[    1.735660] -(0)[194:kworker/u8:3][<c092f3f0>] (gt1x_irq_enable) from [<c092fb48>] (tpd_i2c_probe+0x1e0/0x2f4)
[    1.735714] -(0)[194:kworker/u8:3] r5:00000000 r4:c17f9ba0
[    1.735756] -(0)[194:kworker/u8:3][<c092f968>] (tpd_i2c_probe) from [<c0939870>] (i2c_device_probe+0x1d8/0x20c)
[    1.735811] -(0)[194:kworker/u8:3] r8:c092f968 r7:cf748a04 r6:cf748a20 r5:cf748a00
[    1.735877] -(0)[194:kworker/u8:3][<c0939698>] (i2c_device_probe) from [<c053cedc>] (driver_probe_device+0x318/0x470)