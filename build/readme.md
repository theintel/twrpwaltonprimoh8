This will consist of all the unsuccessful and successful builds.

 "Build 0"
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