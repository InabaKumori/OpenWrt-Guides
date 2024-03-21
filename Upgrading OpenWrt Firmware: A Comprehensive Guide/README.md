Upgrading OpenWrt Firmware: A Comprehensive Guide

Introduction:
Upgrading your OpenWrt firmware is an essential process to keep your router up-to-date, secure, and functioning optimally. This comprehensive guide will walk you through the entire process, from preparation to post-upgrade steps, ensuring that you retain your settings, installed packages, and data. While this guide aims to be as detailed as possible, it's crucial to understand that unexpected issues can still arise. Therefore, creating a backup of your router's configuration is highly recommended.

Prerequisites:
1. Compatible OpenWrt firmware for your device
2. Access to your router's web interface (LuCI) or SSH access
3. List of installed packages (optional but recommended)

Step 1: Determine Your Router's Architecture and Current Firmware
1. Connect to your router via SSH or access the LuCI web interface.
2. To determine your router's architecture, run the following command in the SSH terminal:
   ```
   uname -m
   ```
   This will output your router's architecture (e.g., x86_64, aarch64, etc.).
3. To check your current firmware type, run:
   ```
   [ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"
   ```
   If the output is "UEFI," your router uses UEFI firmware. If it's "BIOS," your router uses BIOS firmware.
4. Note down your router's architecture and firmware type for later reference.

Step 2: Download the Appropriate Firmware
1. Visit the official OpenWrt download page: https://downloads.openwrt.org
2. Navigate to the directory corresponding to your router's architecture (e.g., x86_64, aarch64).
3. Locate the appropriate firmware image based on your router's architecture and firmware type. The naming convention is as follows:
   - generic-ext4-combined-efi.img.gz: UEFI firmware with ext4 filesystem
   - generic-ext4-combined.img.gz: BIOS firmware with ext4 filesystem
   - generic-squashfs-combined-efi.img.gz: UEFI firmware with SquashFS filesystem
   - generic-squashfs-combined.img.gz: BIOS firmware with SquashFS filesystem
4. Download the selected firmware image to your computer.

Step 3: Backup Your Current Configuration
1. Via LuCI (Web Interface):
   - Navigate to "System" > "Backup / Flash Firmware".
   - Click on "Generate archive" to download a backup of your current settings.
2. Via SSH:
   - Connect to your router via SSH.
   - Run the following command to create a backup archive:
     ```
     sysupgrade -b /tmp/backup-$(date +%Y-%m-%d).tar.gz
     ```
   - Transfer the backup file from `/tmp` to your computer using `scp` or a similar tool.

Step 4: List Installed Packages (Optional)
1. Connect to your router via SSH.
2. Run the following command to list installed packages:
   ```
   opkg list-installed > /tmp/packages-installed.txt
   ```
3. Transfer the `packages-installed.txt` file to your computer for reference after upgrading.

Step 5: Perform the Upgrade
1. Via LuCI:
   - Navigate to "System" > "Backup / Flash Firmware".
   - Under "Flash new firmware image," click "Choose File" and select the firmware image you downloaded.
   - Check "Keep settings" to retain your configurations.
   - Click "Flash image" to proceed with the upgrade.
   - Review the file and checksum confirmation page, then confirm to start the flashing process.
2. Via SSH:
   - Transfer the downloaded firmware to your router's `/tmp` directory using `scp` or a similar tool.
   - Connect to your router via SSH.
   - Run the following command to initiate the upgrade, replacing `<firmware-file>` with the name of the firmware image:
     ```
     sysupgrade -v /tmp/<firmware-file>
     ```
   - Ensure you've used the correct command flags to keep your settings, as specified in the OpenWrt documentation.

Step 6: Resize the Root Filesystem (If Necessary)
1. After the upgrade, if you encounter insufficient space issues, you may need to resize the root filesystem.
2. Connect to your router via SSH.
3. Run the following command to determine your root filesystem type:
   ```
   df -Th
   ```
   Look for the filesystem mounted on `/` (root). It will be either `ext4` or `squashfs`.
4. Follow the appropriate steps based on your filesystem type:
   
   Resizing OpenWrt System Root Filesystem: A Comprehensive Guide
   [Parent Directory File](../OpenWrt-Guides/"Resizing OpenWrt System Root Filesystem: A Comprehensive Guide"/README.md)
   
6. After the reboot, verify that the root filesystem has been resized by running `df -h`.

Step 7: Add Custom Repositories (Optional)
1. Connect to your router via SSH.
2. Navigate to the `/etc/opkg/` directory.
3. Open the `customfeeds.conf` file using a text editor (e.g., `vi`).
4. Add the following lines to include recommended custom repositories:
   ```
   src/gz kuoruan_universal http://openwrt.kuoruan.net/packages/releases/all
   src/gz v2raya https://downloads.sourceforge.net/project/v2raya/openwrt/x86_64
   src/gz openwrt_kiddin9 https://dl.openwrt.ai/23.05/packages/x86_64/kiddin9
   ```
5. Save the file and exit the editor.
6. Update the package lists by running:
   ```
   opkg update
   ```
7. If you encounter key verification errors, you can temporarily disable key verification by running:
   ```
   opkg --conf /etc/opkg.conf --no-check-certificate update
   ```
   Note: Disabling key verification is not recommended for regular use as it compromises security.

Step 8: Reinstall Packages (If Necessary)
0. (Optional) To install all packages listed in a `packages-installed.txt` file on an OpenWrt system while ensuring the system remains stable and avoids overloading, follow these steps meticulously. Before proceeding with the mass installation of packages, it's crucial to prepare the system by updating package lists and installing a necessary utility like `screen`. This utility allows commands to continue running even if the SSH session is interrupted, reducing the risk of incomplete installations.
1. If you had previously installed packages that are not included in the new firmware, you can reinstall them using the `packages-installed.txt` file from Step 4.
2. Connect to your router via SSH.
3. Run the following command to install all packages listed in the file:
   ```
   opkg install $(cat /root/packages-installed.txt)
   ```
   Note: Replace `/root/packages-installed.txt` with the actual path to your `packages-installed.txt` file.
4. If you encounter any errors during the installation process, try running the command again. Some packages may have dependencies that need to be installed first.

Step 9: Post-Upgrade Verification
1. After the upgrade and package reinstallation (if applicable), verify that your settings have been retained and the router is functioning correctly.
2. Test your network connectivity, firewall rules, and any custom configurations you had previously set up.
3. If you encounter any issues, refer to the OpenWrt documentation or seek assistance from the OpenWrt community forums.

Conclusion:
Upgrading your OpenWrt firmware requires careful planning and execution to ensure a smooth transition while retaining your settings and data. By following this comprehensive guide and creating backups at crucial steps, you can minimize the risk of losing important configurations. Remember to always download firmware from official sources, verify compatibility with your device, and exercise caution when adding custom repositories. With patience and attention to detail, you can successfully upgrade your OpenWrt router and enjoy the benefits of the latest features and security enhancements.
