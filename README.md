# Overviev
  This guide was created to help anyone who wants to create a bootable ssd or hdd with linux that is usable on any computer.
  I spent a long time researching how to do this as well as fixes for problems I encountered and this will walk you through the entire    process.
  This guide uses the ubuntu operating system so commands may vary for other operating systems.
  This guide will also make the drive compatable with Apple computers.

## Installing Linux 
  This is the first step to making the bootable drive and requires you to have both the ssd/hdd you would like to use as well as a usb to save the ubuntu ISO to.
  Download the Ubuntu ISO to your system from https://ubuntu.com/download/desktop. It is ok to get the older version at this stage as it will be updated later anyway.
    
  * Insert both the usb and ssd into your ssd and open a terminal to execute the following commands.
  * These commands are for macOS specifically so you will need to look online for windows equivilents.
      ```
      diskutil list
      diskutil eraseDisk FAT32 UBUNTU /dev/<YOUR DISK ID> 
      diskutil unmountDisk /dev/<YOUR DISK ID>
      ```
  * Repeat those three commands for both the USB and SSD/HDD.
        
  * The following command should be only executed on the USB. Take extra care when executing this command as you dont want to do it incorrectly.
      ```
      sudo dd if=<PATH TO UBUNTU LIVE ISO> of=/dev/<YOUR USB DISK ID>
      ```
   * Once this is done you can power off the system with both the usb and ssd/hdd still plugged in. 
        
## Setting up the partitions 
  This step will walk you through the process of installing linux onto the main ssd/hdd and setting up the partitions.
  
  * Power on your machine and repeatedly press the f12 button(could be another button depending on your motherboard manufacturer) until you come to a set of options in a boot menu.
  * You want to click the option that is for your USB. Look for a line names UEFI <USB NAME> and double click it.
  * When a ubuntu menu appears click the first option to test out ubuntu first.
    
  * When you load in you should see a desktop with a series of applications on the left. Look for one called install Ubuntu and click it.
  * You need to go through this process up until you get to a page that will ask you what you want to install. Use the top option that is the reccommended install and you may optionally select any other options below it. 
  * When you press continue after this there will be a short wait and you should then be brought to a menu regarding where you would like to install Ubuntu to. Make sure to only select the option at the bottom called "Something Else".
  * When you get to the next page you should see a series of drives most likely titled sda, sdb, sdc and so on. Find the one that is corresponding to your ssd/hdd and click it and then click create new partition. 
  * You should be left with one unallocated of the size of your ssd.
  * Click this and press the small + sign at the bottom left to create your first partition.
    
  * The first partition you want to make is the swap partition. Set the size of the partition to be equal to the amount of ram your computer has. 
  * Select the bottom two options of each section and finally select the partition format to swap area and then save.
    
  * The second partition you want to make is the main storage. Click the + again and leave all the settings as is except for the format which should be set to ext4 and the directory to /.
    
  * You then need to go to the bottom of that window and where you see a drop down menu of drives, select the USB that has the Ubuntu ISO loaded on. Then click install.
    
  * Continue on through these options untill you finish the installation and are instructed to reboot the system. When this happens again select the USB as the option to boot from and select the test ubuntu first option.
    
## Configuring the ssd/hdd
  At this stage you have a functioning bootable ssd/hdd however at the moment you are only going to be able to use it on the system you created it on. This needs to be fixed.
    
  * Start by opening the terminal and running ```sudo fdisk -l``` to get a list of partitions.
  * Identify the one that has the linux partitions. In my case this is /dev/sdb. For the purposes of this guide we will call that /dev/sdX.
  * You also need to identify the partition that contains the root filesystem. In my case this is /dev/sdb2. We will refer to it as /dev/sdXY from now on.
  * Launch GParted from the terminal with ```sudo gparted /dev/sdX```
    
  * From here you need to resize your main root partition to allow for 200mb more space elsewhere. Right click the root partition and select unmount. From there right click it again and click resize. Where you see the large number representing your size ssubtract 200 away from that number and click save. 
      When you do this you should then see another new partition below it which is exactly 200mb in size.
      Createa new partition in that space and change the file system to fat32. Then click the apply operations button at the top of the window. 
      After this applies, right click the new partition you just made and click manage flags.  Set the flags to boot and esp. From now on I will refer to this partition /dev/sdXZ.
    
  * We now have a dedicated partition for the ESP however we need to make sure the ubuntu installation on the ssd/hdd can see it. 
  
  * Launch Gparted from the terminal with ```sudo gparted /dev/sdX```
  * Double click the linux root filesystem and write down the UUID so you can refer to it later. 
  * Double click the new fat32 partition you just made and note down its UUID as well. This UUID should be much shorter than the previous one. 
  * Close gparted and reopen the terminal.
    
  * Run the following commands
      ```
      sudo umount /media/ubuntu/<UUID you just wrote down from the linux root filesystem>
      sudo mount /dev/sdXY /mnt
      
      sudo nano /mnt/etc/fstab
        Be very careful with this file as you dont want to mess anything up.
        When you open this file you should see a line which contains /boot/efi. Comment out this line by placing a # infront of it. 
        Add the following line at the bottom of that document 
        UUID=<UUID you copied down from the new partition. Should be 4 characters followed by a - then 4 more characters> /boot/efi vfat defaults 0 1
      - Save that file by pressing ctrl + x then press Y and enter.
      ```
  * We have now created a ESP however we need to install GRUB2 on it.
  * In terminal execute the following commands.
        ```
        sudo mount /dev/sdXZ /mnt/boot/efi
        for i in /dev /dev/pts /proc /sys; do sudo mount -B $i /mnt/$i; done
        sudo cp /etc/resolv.conf /mnt/etc/
        modprobe efivars
        sudo chroot /mnt
        grub-install -d /usr/lib/grub/x86_64-efi --efi-directory=/boot/efi --removable /dev/sdX
        ```
  * Your ssd/hdd is now completely bootable on any system however some optional things can be done which I will outline below.
  
Even though the ssd/hdd will boot on any system, it will not yet be compatable with an apple computer. 
The  solution to this is to update ubuntu to the latest version which will improve the system both from the updates and if you happen to be an apple user by allowing you to use it on an apple computer. Only do this if you are ok with not using the LTS version of linux or you want to use a apple computer.

## Updating Ubuntu - 
  The current version of ubuntu will be 18.04. We first need to upgrade this to 19.04 then finally to 19.10.
  
  * Start by running the following commands
    ```
    sudo apt update && sudo apt dist-upgrade
    sudo apt install update-manager-core
    sudo nano /etc/update-manager/release-upgrades
      When this file opens change the bottom prompt value from lts to normal then save and exit the file
    do-release-upgrade
      This should trigger the upgrade so follow any on screen instructions and accept any pop ups that occur. When finished reboot the system.
     ```
  * You now have a system running Ubuntu 19.04
  * This will still not run well on macs however so you must now update it to 19.10
  
  * Open the Software & Updates application
  * Select the 3rd tab called Updates
  * Set the notify me of a new Ubuntu version dropdown menu to for any new version
  * Open a terminal and type ```update-manager -c -d```
  * This should open the application and allow you to install the new version. If however it does not then you have the same problem I did and need to run another final command.
      ```/usr/lib/ubuntu-release-upgrader/check-new-release-gtk```
  * Click upgrade the follow it along and reboot.
    
You should now have a fully bootable version of linux usable on any system.
