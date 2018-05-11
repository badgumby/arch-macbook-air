# Installing Arch Linux on a 2017 Macbook Air
Install instructions for Macbook Air OSX and Arch side by side

## Getting Started
Before starting you should have some knowledge of partitioning and Arch Linux.
Also, I'd like to note that, more than likely, the wireless card in your Macbook Air will not be detected by the Arch boot disk (or most other distros). With that being said, you will need a USB to Ethernet adapter. I like [this one](https://www.amazon.com/AmazonBasics-1000-Gigabit-Ethernet-Adapter/dp/B00M77HMU0/ref=sr_1_3?ie=UTF8&qid=1526041709&sr=8-3&keywords=usb+ethernet).

## Install Instructions
1. You will first need to resize your MacOS partition. You can do this using the Mac Disk Utility. If you are using APFS(Encrypted), you will be required to reinstall MacOS first, since it the encrypted file system will not allow a resize.
2. Turn off System Integrity Protection (SIP).

   + Restart your MacOS
   + Before it starts, press and hold Command+R
   + Select `Terminal` from the `Utility` menu
   + In the terminal, enter `csrutil disable`
   + You should see a message similar to `Successfully disabled System Integrity Protection. Please restart the machine for the changes to take effect.`
   + Reboot your machine

3. Download and install [rEFInd](https://sourceforge.net/projects/refind/).

   + Download the zip
   + Extract to a folder
   + In `Terminal` enter the directory that was extracted
   + Run the command `sudo ./`
