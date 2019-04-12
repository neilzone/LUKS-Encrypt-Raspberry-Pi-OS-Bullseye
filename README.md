# Luks-Encrypt-Raspbian-Stretch
Scripts to luks encrypt the root partition of a Raspbian installation  
Thank you styper.  
I have not tried this yet nor have I looked at the scripts but I am very excited because this should allow the use of Stretch with LUKs Full Disk Encryption whereas before we were limited to staying with Jessie.  
I will report back here after trying this on Stretch.  
[Directions by styper were found here](https://www.raspberrypi.org/forums/viewtopic.php?t=219867)  
The directions are reproduced below and modified to address a www.privatekeyvault.com specific installation.  

### Directions:  
The tutorials styper referenced are the following:  
* [This is where it all started](https://robpol86.com/raspberry_pi_luks.html)  
  * robpo figured out a way to encrypte an SD card using only the raspberry pi whereas before you had to stick your SD card into another type of computer in order to accomplish the job.  
  * Using only the pi for encryption reduces the chances of picking up a malware infection.  
  * And now poor people who only have a raspberry pi have access to full disk encryption.  
* [My tutorial](https://github.com/johnshearing/PrivateKeyVault#setup-luks-full-disk-encryption) made from robpo's was I hope easier to follow and focused toward the PrivateKeyVault. Both these tutorials were for Raspbian Jessie and would not work on Stretch.  


What you need:  
Styper calls for a Raspberry PI 3 but a regular PI 3 can not be used in the PrivateKeyVault because the vault is airgapped whereas the PI 3 is WiFi and Bluetooth enabled. The PI 2 does not have any radios on board which is why I use it in the Vault. In a future build of the Vault I would like to use the Raspberry Pi Compute Module 3+/Lite which has the fast processor of the PI 3 without the radios. In any case, I can't imagine why styper's scripts wouldn't work on a PI 2. I will report back here.  

An sdcard with Raspbian Stretch installed (styper used the lite edition in tests)  
A flash drive connected to the RPI (needed to copy the data from root partition during encrypt so you don't lose it)  

This tutorial should be usable with an already running Raspbian Stretch, just skip the burning iso/img part  

Burn the Raspbian Stretch image to the SDCard using Etcher or a similiar tool  

Download the scripts from the repo and place them inside `/boot/install/`    

Run script: `sudo /boot/install/1.update.sh`  
What this does is update the system. In styper's first try there was a bug with a kernel version that was sending the system into a kernel panic during the process. He/she reports that didn't happened when he/she updated to 4.14.  

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
The script copies all your data to the flash drive because Luks deletes everything when it's encrypting the partition.  
When luks encrypts the root partition it will ask you to type `YES` (in uppercase) then the decryption password twice.   
**watch out if you used CAPS LOCK to type the YES**  
So add a new **strong** password to your liking.  
Then Luks will ask for the decryption password again so we can copy the data back from the flash drive to the root partition

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
There is probably an easier way to do this using chroot so you don't need to reboot so much but I don't know how to do it yet.  
I added expect to the initramfs hook because I'll probably add another script to auto generate a strong password, it can be removed though.  
