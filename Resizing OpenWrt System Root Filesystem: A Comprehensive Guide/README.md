# Resizing OpenWrt System Root Filesystem: A Comprehensive Guide

## Introduction

This guide is designed for users who have some experience in general computing and system administration but are relatively new to OpenWrt. It provides a step-by-step approach to resizing the root filesystem on an OpenWrt device, ensuring that even beginners can follow along without any issues. The guide is based on the relevant sections of the official OpenWrt on x86 Hardware page: https://openwrt.org/docs/guide-user/installation/openwrt_x86

## Understanding the Root Cause

OpenWrt originated outside the x86 ecosystem, resulting in a unique distribution method. Instead of using installation media or an installer, the installation image is copied directly to the target system's hard drive or SSD. Consequently, a new installation of OpenWrt 22.x.x typically has a boot drive with approximately 120 MB of total space, regardless of the actual drive capacity. To reclaim the remaining capacity, you can either resize the root partition or create additional partitions. This guide focuses on resizing the root partition.

## Identifying Your System Configuration

Before proceeding, it's crucial to determine the type of system you have and the image used for installation. OpenWrt offers four mainstream images for x86 hardware:

1. `generic-ext4-combined-efi.img.gz`: Produces a UEFI system with an ext4 file system on the root partition.
2. `generic-ext4-combined.img.gz`: Produces a BIOS system with an ext4 file system on the root partition.
3. `generic-squashfs-combined-efi.img.gz`: Produces a UEFI system with a SquashFS file system on the root partition.
4. `generic-squashfs-combined.img.gz`: Produces a BIOS system with a SquashFS file system on the root partition.

The firmware type (BIOS vs. UEFI) and the root partition file system (ext4 vs. SquashFS) impact the resizing process.

## Resizing Steps for BIOS Systems

For BIOS systems, resizing the root partition involves two steps:

1. Resize the root partition (same process for ext4 and SquashFS).
2. Resize the root file system (different process for ext4 and SquashFS).

## Resizing Steps for UEFI Systems

UEFI systems require an additional step because UEFI keeps track of partition identifiers. When resizing the root partition, its ID changes, and you need to update the UEFI configuration. The resizing process for UEFI systems involves four steps:

1. Capture the initial partition configuration (same process for ext4 and SquashFS).
2. Resize the root partition (same process for ext4 and SquashFS).
3. Update the UEFI configuration file (`/boot/grub/grub.cfg`) (same process for ext4 and SquashFS).
4. Resize the root file system (different process for ext4 and SquashFS).

## Safety Considerations

Resizing partitions and file systems on live drives carries a risk of file system corruption. Ideally, you should perform these operations with the relevant drive unmounted by booting the device from a USB stick. However, in practice, resizing works in most cases, especially when the device is idle.

## Preparations

Before starting, ensure you have the necessary utilities installed. Update the package list and install the required packages:

```
opkg update && opkg install lsblk fdisk losetup resize2fs
```

## Diagnostics

If you're unsure about the image used for installation, you can perform two simple tests to determine the file system type and firmware type.

1. Identifying the File System Type:
   Run the following command to list the file systems with their types:
   ```
   df -Th
   ```
   Look for a file system named `/dev/root`. Its type will be either `ext4` or `squashfs`.

2. Identifying the Firmware Type (BIOS vs. UEFI):
   Run the following command:
   ```
   ls /sys/firmware/efi
   ```
   If the directory `/sys/firmware/efi` exists, your system is UEFI. If you get an error message saying "No such file or directory," your system is BIOS.

## Capturing the Current Partition Configuration (UEFI Systems Only)

If you have a BIOS system, skip this section and proceed to the "Resizing the Root Partition" section.

For UEFI systems, run the following command to display the current partition layout, including each partition's unique identifier:

```
lsblk -o PATH,SIZE,PARTUUID
```

Save the output for future reference. If you're working over SSH, save it on the administrator's computer by copying it from the terminal and saving it as a text file. If you have a monitor and keyboard connected to your device, save it into a text file on the device. For example:

```
lsblk -o PATH,SIZE,PARTUUID > /root/lsblk_old.txt
```

As a precaution, you may want to copy this file to a USB stick.

Next, review the UEFI configuration file:

```
cat /boot/grub/grub.cfg
```

Look for instances where the partitions' unique identifiers are used. You will need to replace them later when updating the UEFI configuration.

## Resizing the Root Partition

The process for resizing the root partition is nearly identical for all cases. Follow these steps:

1. Start an interactive `fdisk` session:
   ```
   fdisk /dev/sda
   ```

2. Print the current partition layout by entering `p` at the command prompt.

3. Delete the existing root partition by entering `d` at the command prompt. When prompted for the partition number, enter the number corresponding to the root partition (usually `2`).

4. Create a new partition by entering `n` at the command prompt. When prompted for the partition type, select `p` for primary (if asked). For the partition number, choose the same number as the deleted partition (usually `2`). When prompted for the first sector, enter the same value as the start sector of the deleted partition. For the last sector, press Enter to use the default value, which will extend the partition to the end of the disk.

5. If prompted to remove the signature, enter `n` to keep the existing signature.

6. Write the changes to the disk by entering `w` at the command prompt.

7. Verify that the partition has been resized by running `fdisk -l` or starting another interactive session (`fdisk /dev/sda`) and using the `p` command to display the partition layout, followed by the `q` command to quit.

## Updating the UEFI Configuration (UEFI Systems Only)

If you have a BIOS system, skip this section and proceed to the "Resizing the Root File System" section.

For UEFI systems, follow these steps:

1. Run the following command to display the updated partition layout:
   ```
   lsblk -o PATH,SIZE,PARTUUID
   ```

2. Compare the output with the earlier version you saved. Note the new unique identifier for the `/dev/sda2` partition.

3. Open the UEFI configuration file in a text editor. For example, using `vi`:
   ```
   vi /boot/grub/grub.cfg
   ```

4. Replace the old partition identifier with the new one in all the necessary places within the configuration file.

5. Save the changes and exit the text editor.

## Resizing the Root File System

The process for resizing the root file system differs depending on whether you have an ext4 or SquashFS file system. Execute the commands one by one, allowing each command to finish before entering the next one.

For ext4 Systems:

```
BOOT="$(sed -n -e "\|\s/boot\s.*$|{s///p;q}" /etc/mtab)"
PART="${BOOT##*[^0-9]}"
DISK="${BOOT%${PART}}"
ROOT="${DISK}$((PART+1))"
LOOP="$(losetup -f)"
losetup ${LOOP} ${ROOT}
resize2fs -f ${LOOP}
reboot
```

For SquashFS Systems:

```
BOOT="$(sed -n -e "\|\s/boot\s.*$|{s///p;q}" /etc/mtab)"
PART="${BOOT##*[^0-9]}"
DISK="${BOOT%${PART}}"
ROOT="${DISK}$((PART+1))"
LOOP="$(losetup -n -l | sed -n -e "\|\s.*\s${ROOT#/dev}\s.*$|{s///p;q}")"
resize2fs -f ${LOOP}
reboot
```

Watch the output of the `resize2fs` command. Normally, you should see an indication of the file system size change. If `resize2fs` indicates that it has nothing to do, reboot your device and repeat this step in its entirety. Rebooting ensures that the changes made during the previous steps are applied to the system.

## Verifying the Resizing Process

After completing the resizing process, it's important to verify that everything worked as expected. Follow these steps:

1. Reboot your device to ensure that all changes take effect:
   ```
   reboot
   ```

2. Once the device has restarted, log in and check the available disk space:
   ```
   df -h
   ```
   Look for the root file system (usually mounted at `/`) and verify that its size has increased according to your expectations.

3. Optionally, you can run `fdisk -l` to display the updated partition layout and confirm that the root partition has been resized correctly.

## Troubleshooting

If you encounter any issues during the resizing process or after rebooting, consider the following troubleshooting steps:

1. Double-check that you followed all the steps correctly, especially the commands for resizing the root partition and updating the UEFI configuration (if applicable).

2. If the device fails to boot after resizing, try booting into failsafe mode. This can often be done by selecting the "OpenWrt (failsafe)" option from the boot menu or by pressing a specific key during the boot process (refer to your device's documentation for details).

3. If you can access the device in failsafe mode, review the system logs for any error messages related to the resizing process or file system issues. You can use the `dmesg` command to view the kernel log messages.

4. If the root file system resize operation failed or wasn't applied, try repeating the resizing steps while the device is booted in failsafe mode.

5. If you continue to experience issues, consider seeking assistance from the OpenWrt community forums or IRC channels. Provide as much detail as possible about your device, the steps you followed, and any error messages you encountered.

## Additional Considerations

- If you have sufficient disk space, you can create additional partitions instead of or in combination with resizing the root partition. This can be useful for separating user data or specific services from the main system partition.

- Before performing any major system changes, it's always a good idea to create a backup of your device's configuration and any important data. This allows you to restore the system if something goes wrong during the resizing process.

- If you're using a USB storage device for your OpenWrt installation, be aware that some USB devices may have limited write endurance. Resizing the root partition and file system on such devices frequently may lead to premature wear and potential data loss. Consider using high-quality USB storage devices or, if possible, switch to a more reliable storage medium like an SSD.

- After resizing the root partition, you may want to consider optimizing the file system for better performance. For ext4 file systems, you can use the `tune2fs` utility to adjust various file system parameters. Consult the `tune2fs` manual page (`man tune2fs`) for more information on available options and their effects.

## Conclusion

Resizing the root partition and file system on an OpenWrt device may seem daunting at first, but by following this comprehensive guide, you should be able to complete the process successfully. Remember to take your time, double-check your commands, and create backups whenever possible. If you encounter any issues, don't hesitate to seek help from the OpenWrt community.

With a resized root partition, you'll have more space available for installing additional packages, storing data, or running services on your OpenWrt device. Enjoy the increased flexibility and possibilities that come with a larger root file system!
