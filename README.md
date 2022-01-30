# LUKS-Encrypt-Raspberry-Pi-OS-Bullseye

(Testing using Raspberry PI OS, running on a Raspberry Pi 4.)

## Preparations

You'll need:

- a Raspberry Pi
- a microSD card with the Raspberry Pi OS imaged to it (e.g. via `dd` or Etcher)
- a flash drive of at least the same size as the microSD card. This will be wiped as part of the process 


# 1: Update your Raspberry Pi

Create `/boot/install`:

```
sudo mkdir -p /boot/install
```

Download the scripts to `/boot/install/`:

```
sudo apt install git -y
git clone https://github.com/neilzone/LUKS-Encrypt-Raspberry-Pi-OS-Bullseye /boot/install/
```

Run script: `sudo /boot/install/1.update.sh`  

This updates your Raspberry Pi OS installation.

When it is finished, execute the following command at the bash console:  
`sudo reboot`  
This is needed so the system reboots and loads the new kernel version.  

# 2: Prepare for disk encryption

Log back into your Raspberry Pi, and run script: `/boot/install/2.disk_encrypt.sh`  
This prepares the environment adding new applications to initramfs to make the job easier and prepares the needed files for LUKS.

When it is finished, plug in your USB flash drive.

Then, execute the following command at the bash console:  
`sudo reboot`  


When your Raspberry Pi reboots, it will reboot into the initramfs shell. This may take a while.

# 3: Start the encryption process

In the initramfs shell run the following commands:  
`mkdir /tmp/boot`  
`mount /dev/mmcblk0p1 /tmp/boot/`  
`/tmp/boot/install/3.disk_encrypt_initramfs.sh`  

This copies your microSD card to your flash drive. This is because the LUKS encryption process deletes everything when it is encrypting the partition. When the process is completed, the script will copy it back again to the microSD card.

Be patient: this can take a long time.

When LUKS encrypts the root partition it will ask you to type `YES` (in uppercase). You **must** use uppercase.

You will be asked to choose a decryption password, and enter it twice.   

It will then encrypt your microSD card.

When finished, LUKS will ask for the decryption password again. It unlocks the microSD card, and then copies back the data from the microSD card.

When finished, remove your USB flash drive.

Then, execute the following command at the bash console:  
`reboot -f`  


As before, it will boot into the initramfs shell. This is because it cannot unlock your microSD card's encrypted partition, to boot Raspberry Pi OS.

# 4: Unlock the drive to boot into Raspberry Pi OS

Execute the following commands at the initramfs shell:  
`mkdir /tmp/boot`  
`mount /dev/mmcblk0p1 /tmp/boot/`  
`/tmp/boot/install/4.luks_open.sh`   

Type in your decryption password again.

When it drops back to the initramfs prompt, type `exit`. 

The system should resume booting, and will boot into Raspberry Pi OS.

# 5: Configure so you do not need to boot into initramfs each time

Run script: `/boot/install/5.rebuild_initram.sh`  

This step means you do not need to boot first into initramfs and unlock the drive, before it continues to boot.

# 6: configure unlocking over SSH

Because you have used LUKS, you need to enter your passphrase to reboot your Raspberry Pi.

You can [enable remote unlocking of the microSD card over SSH](https://neilzone.co.uk/2021/06/unlocking-a-luks-encrypted-partition-via-ssh-on-debian-10), before it boots into Raspberry Pi OS.
