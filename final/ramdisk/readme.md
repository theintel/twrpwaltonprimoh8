Files directly copied from stock ramdisk and renamed appropriately:
renamed to ueventd.rc from ueventds.rc
fstab.enableswap from fstabs.enableswap

ueventd.rc sets permissions for device drivers. So ueventds.rc must replace its counterpart ueventdt.rc in twrp ramdisk.

fstab.enableswap is nonexistent in twrp ramdisk but is associated with swap memory in stock device.