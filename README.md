# Installing Arch Linux on a 2017 Macbook Air
Install instructions for Macbook Air OSX and Arch side by side

## Getting Started
Before starting you should have some knowledge of partitioning and Arch Linux.
Also, I'd like to note that, more than likely, the wireless card in your Macbook Air will not be detected by the Arch boot disk (or most other distros). With that being said, you will need a USB-to-Ethernet adapter. I like [this one](https://www.amazon.com/AmazonBasics-1000-Gigabit-Ethernet-Adapter/dp/B00M77HMU0/ref=sr_1_3?ie=UTF8&qid=1526041709&sr=8-3&keywords=usb+ethernet).

Items needed for install:

   + Internet connection
   + USB-to-Ethernet Adapter
   + USB flash drive for the bootable .dmg (covered later)

## Install Instructions
01. You will first need to resize your MacOS partition. You can do this using the Mac Disk Utility. If you are using APFS(Encrypted), you will be required to reinstall MacOS first, since the encrypted file system will not allow a resize.
02. Turn off System Integrity Protection (SIP).

   + Restart your MacOS
   + Before it starts, press and hold `Command+R`
   + Select `Terminal` from the `Utility` menu
   + In the terminal, enter `csrutil disable`
   + You should see a message similar to `Successfully disabled System Integrity Protection. Please restart the machine for the changes to take effect.`
   + Reboot your machine

03. Download and install [rEFInd](https://sourceforge.net/projects/refind/).

   + Download the zip
   + Extract to a folder
   + In `Terminal` enter the directory that was extracted
   + Run the command `sudo ./refind-install`

04. Download the current [Arch Linux iso](https://www.archlinux.org/download/).
05. Convert `archlinux-20xx.xx.xx-x86_64.iso` to `.dmg` format.

   + `hdiutil convert -format UDRW -o archlinux-20xx.xx.xx-x86_64.iso arch-converted.dmg`

06. Write the `arch-converted.dmg` to a USB flash drive.

   + In a terminal, enter `diskutil list`
   + Identify your USB drive. In my case it was `/dev/disk2`
   + Once you have identified the disk, enter `diskutil unmountDisk /dev/disk2` (replace disk2 with your USB drive)
   + Once unmounted, enter `sudo dd if=arch-converted.dmg of=/dev/rdisk2 bs=1m` (use an `r` before disk to access the raw drive)

07. Boot to USB drive.

   + Power down your Macbook
   + Insert your USB-to-Ethernet Adapter
   + Power on, press and hold the `Alt/Option` key
   + On the UEFI selection menu, select your boot USB drive

08. Verify network connectivity.

   + Verify adapter appears and has connetivity. Enter `ip addr`
   + If DHCP has not started on the NIC enter `systemctl start dhcpcd@INTERFACE`, where `INTERFACE` is the NIC name.
   + Ping Google to verify internet connection. Enter `ping 8.8.8.8`

09. List devices and format unused partition.

   + Enter `fdisk -l` to list the disks
   + Enter `sgdisk -n 0:0:0 -t 0:8300 -c 0:"data" /dev/sda3` (This assumes the partition you created is on device `sda` and is partition `3`)
   + Inform OS of partition changes by entering `partprobe /dev/sda` (If device is not `sda`, enter correct device name)
   + Make the filesystem format using `mkfs.ext4 /dev/sda3`

10. Mount the newly created partition.

   + `mount /dev/sda3 /mnt`

11. Create a swap file.

   + Create the file: `dd if=/dev/zero of=/mnt/swapfile bs=1M count=2048` (Change count to how big you would like it. I have it set to be 2GB)
   + Change permissions on the file: `chmod 600 /mnt/swapfile`
   + Inform OS to use as swap: `mkswap /mnt/swapfile`

12. Perform initial install of Arch to mounted volume.

   + `pacstrap /mnt base base-devel grub-efi-x86_64 linux-headers broadcom-wl-dkms wpa_supplicant dialog vim git reflector`

13. Generate the `/etc/fstab`.

   + `genfstab -pU /mnt >> /mnt/etc/fstab`
   + `echo 'tmpfs	/tmp	tmpfs	defaults,noatime,mode=1777	0	0' >> /mnt/etc/fstab`
   + `echo '/swapfile none  swap defaults 0 0' >> /mnt/etc/fstab`

14. Enter the fresh installed Arch system.

   + `arch-chroot /mnt /bin/bash`

15. Initialize the `pacman-key`.

   + `pacman-key --init`
   + `pacman-key --populate archlinux`

16. Update timezone information.

   + `rm /etc/localtime`
   + `ln -s /usr/share/zoneinfo/America/Chicago /etc/localtime`
   + `hwclock --systohc --utc`

17. Set hostname.

   + `echo MYHOSTNAME > /etc/hostname`

18. Set your locale.

   + `sed -i 's:#en_US.UTF-8 UTF-8:en_US.UTF-8 UTF-8:g' /etc/locale.gen`
   + `locale-gen`
   + `echo LANG=en_US.UTF-8 >> /etc/locale.conf`
   + `echo LANGUAGE=en_US.UTF-8 >> /etc/locale.conf`
   + `echo LC_ALL=en_US.UTF-8 >> /etc/locale.conf`

19. Set `root` user password.

   + `passwd`

20. Allow `wheel` group in `sudoers` file.

   + `sed -i '/%wheel ALL=(ALL) ALL/c\%wheel ALL=(ALL) ALL'  /etc/sudoers`

21. Add user and set password.

   + `useradd -m -g users -G wheel $MYUSERNAME`
   + `passwd $MYUSERNAME`

22. Add `ext4` to `/etc/mkinitcpio.conf`.

   + `sed -i '/^MODULES=/c\MODULES=(ext4)' /etc/mkinitcpio.conf`

23. Regenerate the initrd image.

   + `mkinitcpio -p linux`

24. Edit `/etc/default/grub`

   + Modify line: `GRUB_CMDLINE_LINUX_DEFAULT="quiet libata.force=1:noncq"`
   + Add line: `GRUB_DISABLE_SUBMENU=y`

25. Make `grub.cfg` and standalone `boot.efi` file.

   + `grub-mkconfig -o boot/grub/grub.cfg`
   + `grub-mkstandalone -o arch.efi -d usr/lib/grub/x86_64-efi -O x86_64-efi --compress=xz boot/grub/grub.cfg`

26. Mount EFI partition and create. (assuming partition /dev/sda1)

   + `mkdir -p /mnt/EFI`
   + `mount /dev/sda1 /mnt/EFI`
   + `mkdir -p /mnt/EFI/arch`
   + `cp /arch.efi /mnt/EFI/arch/arch.efi`

27.  Exit chroot and reboot.

   + `exit`
   + `reboot`

28. You should now be greeted with the rEFInd boot loader, and see an option for your Arch Linux partition.
