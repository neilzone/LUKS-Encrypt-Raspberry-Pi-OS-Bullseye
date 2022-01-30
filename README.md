# Luks-Encrypt-Raspbian-Stretch

An SD card with Raspbian Stretch installed (styper used the lite edition in tests)  
A flash drive connected to the RPI (needed to copy the data from root partition during encrypt so you don't lose it)  

This tutorial should be usable with an already running Raspbian Stretch, just skip the burning iso/img part  

Burn the Raspbian Stretch image to the SD Card using Etcher or a similiar tool  

Download the scripts from the repo and place them inside `/boot/install/`    

Run script: `sudo /boot/install/1.update.sh`  
What this does is update the system. In styper's first try there was a bug with a kernel version that was sending the system into a kernel panic during the process. styper reports that didn't happened when he/she updated to 4.14.  

Execute the following command at the bash console:  
`sudo reboot`  
This is needed so the system loads the new kernel version.  

Run script: `/boot/install/2.disk_encrypt.sh`  
This prepares the environment adding new applications to initramfs to make the job easier and prepares the needed files for Luks

Execute the following command at the bash console:  
`sudo reboot`  
Now we're going to be dropped to the initramfs shell, this is normal

In the initramfs shell run the following commands:  
`mkdir /tmp/boot`  
`mount /dev/mmcblk0p1 /tmp/boot/`  
`/tmp/boot/install/3.disk_encrypt_initramfs.sh`  
The script above copies all your data to the flash drive because Luks deletes everything when it's encrypting the partition.  

When luks encrypts the root partition it will ask you to type `YES` (in uppercase) then the decryption password twice.   
**watch out if you used CAPS LOCK to type the YES**  
So add a new **strong** password to your liking.  
Then Luks will ask for the decryption password again so we can copy the data back from the flash drive to the root partition.  

Execute the following command at the bash console:  
`reboot -f`  
We're dropped again to the initramfs, this is still normal  

Execute the following commands at the bash console:  
`mkdir /tmp/boot`  
`mount /dev/mmcblk0p1 /tmp/boot/`  
`/tmp/boot/install/4.luks_open.sh`   

Type in your decryption password again, then the system should resume booting as normal, at this point all the data is encrypted already, we just need to rebuild the initramfs.  

Run script: `/boot/install/5.rebuild_initram.sh`  

There it is, once you reboot it will ask for the decrypt password again every time now.  

Some notes:  
* There is probably an easier way to do this using chroot so you don't need to reboot so much but I don't know how to do it yet.  
* I added expect to the initramfs hook because I'll probably add another script to auto generate a strong password, it can be removed though.  
