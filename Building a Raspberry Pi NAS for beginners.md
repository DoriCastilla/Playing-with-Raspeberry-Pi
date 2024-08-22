# **How to build a Raspberry Pi NAS. Guide for beginners**

## Objective
To install and configure a Raspeberry Pi NAS ([Network Attached Storage](https://ca.wikipedia.org/wiki/Network-attached_storage)) in a SOHO LAN
## Steps
1. [Prepare the Raspberry Pi image](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-1-prepare-the-raspberry-pi-image)
2. [Find the IP of the RPi](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-2-find-the-IP-of-the-RPi
)
3. [Access to your RPi in remote](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-3-access-to-your-RPi-in-remote)
4. [Set a static IP and install and config Samba](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-4-Set-a-static-IP-and-install-and-config-Samba)
5. [Check the storage disk is already mounted](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-5-Check-the-storage-disk-is-already-mounted)
6. [Create the folder you will share](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-6-Create-the-folder-you-will-share)
7. [Configure Samba software](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-7-Configure-Samba-software)
8. [Maping Samba drive to access the network folder](https://github.com/DoriCastilla/Playing-with-Raspeberry-Pi/blob/main/Building%20a%20Raspberry%20Pi%20NAS%20for%20beginners.md#step-8-Maping-Samba-drive-to-access-the-network-folder)
## Tools 
- Computer 
- Windows 11
- Raspeberri Pi 3b+ (RPi from now)
- SD card 16GB (it is what I used in this case, but it can be a USB stick also) for the Operative System
- An SD card reader if not in your computer.
- Hard disk or USB for storage
- [RPi image](https://www.raspberrypi.com/software/) software 
- [PuTTY](https://www.putty.org/) software  
- Ethernet  
- Screen   
- Keyboard
- Mouse
## Configuration (used in this exercise)
- User id (uid): pi
- Password (pw): pi
- Static IP: 192.168.1.113
- Gateway: 192.168.1.1
- Subnet: 255.255.255.0

## STEP 1: Prepare the Raspberry Pi image
Download the RPi OS image software [here](https://www.raspberrypi.com/software/).
For this guide I did use **Imager version 1.8.5** downloaded for Windows.<br>
Open the program and install it.

![raspberry_1](https://github.com/user-attachments/assets/27107e8c-2acb-46cb-8ada-f48bfb89f89c)

Plug in the SD card in your card reader on your computer, open the image, install it, and follow the instructions. 
 
Select the settings. For this case:
- RPi Device: **RASPBERRY PI 3**
- Operating system: **RASPBERRY PI OS 64BIT**
- Storage: **SD CARD**  

Press `NEXT`.

Select `EDIT SETTINGS` and set your information. In this case: 

![raspberry_2](https://github.com/user-attachments/assets/a9bb1470-75cd-428a-a26a-6c337d5d15bc)

- Set username and password of your system (in this case: (uid) pi : (pw) pi).
- Add your wifi information for connect to the NAS via wifi.
- In "SERVICES" Select "enable SSH".

![raspberry_3](https://github.com/user-attachments/assets/b1df8177-7d0b-4207-a723-7ef3360b87e3)

Select `SAVE` and `YES`. 

![raspberry_4](https://github.com/user-attachments/assets/ea20d1e3-c330-4c86-ab94-5e1fd4471b0a)

When the software informs you the SD card will erase and ask for continue, select `YES` and ignore pop-up messages that can come out.

![raspberry_5](https://github.com/user-attachments/assets/ac37e08a-d749-4d0b-95b3-6a12ba4cd732)

Once it is completed, with the writing and verifying (it will take some minutes), remove the SD card/USB from your computer and set it into the RPi.  
Connect the hard disk storage and the peripherals (keyboard, screen, and mouse) to the RPi. <br>
I also connected my RPI to the internet by Ethernet. <br>
Turn the RPi on and wait, it will take a few minutes to get ready.

## STEP 2: Find the IP of the RPi
Options:
1. Via router DHCP leases, look for the RPI system name, you gave the system in the image you installed in the SDHC card.
2. Via router by the MAC address in the list of connected devices on the router settings.
3. Open the terminal and write the command (your IP is most likely a number like 192.168.xxx.xxx):  
```
ifconfig
```
![raspberry_6](https://github.com/user-attachments/assets/480de214-797b-41a9-8162-9914983017f1)

## STEP 3: Access to your RPi in remote
When you connect remotely by [PuTTY](https://www.putty.org/) you will not need the peripherals anymore.

Open PuTTy.
Hostname: the IP of your RPi.
```192.168.1.116```
Connection tipe: SSH<br>
Press `Open`
![raspberry_7](https://github.com/user-attachments/assets/f1cbcd48-7111-4d22-af7c-73e86c3ca078)

The PuTTy command line interface will open. <br>
Probably, it will prompt a warning of potential security breach, just `Accept` it.<br>
Login with your user and password. <br>
In this case: <br>
uid: pi <br>
pw: pi

![raspberry_8](https://github.com/user-attachments/assets/584f5dc2-c1c4-4d09-8f6d-445b61185da6)

## STEP 4: Set a static IP and install and config Samba
Update and upgrade the system (paste with right button of the mouse): 
```
sudo apt update && sudo apt upgrade -y
```
![raspberry_9](https://github.com/user-attachments/assets/49d7e708-5076-4b39-8c66-98b313ab361d)


Set the static IP and install and config Samba in the staticip.sh editing the file: 
```
nano staticip.sh
```
And write down this script and edit it with your information where needed:
- username and password you set (in this case: pi / pi)
- static IP you choose (in this case: 192.168.1.113)
- gateway and DNS (in this case is the same)
- the owner (in this case: pi) and
- wifi "no" or "yes" depending if you want to use it (in this case I chose no)
  
```
#!/bin/bash

# Set variables
USERNAME="pi" # Change this to your username 
PASSWORD="pi" # Change this to your password
STATIC_IP="192.168.1.113" # Change this to your desired static IP
GATEWAY="192.168.1.1" # Change this to your gateway IP
DNS="8.8.8.8" # Change this to your preferred DNS
SHARED_DIRECTORY="/srv/samba/share" # Specify your share name for your storage
SHARED_DIRECTORY_PERMISSIONS="0777"
SHARED_DIRECTORY_OWNER="pi:pi" # Change this to your desired owner
ETH_INTERFACE="eth0"
WIFI_INTERFACE="wlan0"
USE_WIFI="no" # Change this to "yes" if you are using WiFi

# Function to set static IP for Bookworm 64-bit
set_static_ip_bookworm() {
    echo "Setting static IP for Bookworm 64-bit..."
    cat <<EOF | sudo tee /etc/network/interfaces.d/$INTERFACE
auto $INTERFACE
iface $INTERFACE inet static
    address $STATIC_IP
    netmask 255.255.255.0
    gateway $GATEWAY
    dns-nameservers $DNS
EOF
    echo "Static IP set for Bookworm 64-bit."
}

# Function to set static IP for 32-bit OS
set_static_ip_32bit() {
    echo "Setting static IP for 32-bit OS..."
    sudo cp /etc/dhcpcd.conf /etc/dhcpcd.conf.backup

    cat <<EOF | sudo tee /etc/dhcpcd.conf
interface $INTERFACE
static ip_address=$STATIC_IP/24
static routers=$GATEWAY
static domain_name_servers=$DNS
EOF

    # Restart the dhcpcd service
    sudo systemctl restart dhcpcd
    echo "Static IP set for 32-bit OS."
}

# Determine the correct interface to use
if [ "$USE_WIFI" == "yes" ]; then
    INTERFACE=$WIFI_INTERFACE
else
    INTERFACE=$ETH_INTERFACE
fi

# Check OS version and architecture and configure static IP accordingly
os_version=$(lsb_release -c | awk '{print $2}')
architecture=$(uname -m)

if [[ "$os_version" == "bookworm" && "$architecture" == "aarch64" ]]; then
    set_static_ip_bookworm
else
    set_static_ip_32bit
fi

# Update and install samba
echo "Updating system and installing Samba..."
sudo apt-get update
sudo apt-get install samba samba-common-bin -y
 
# Create a new user
echo "Creating new user..."
sudo adduser --disabled-password --gecos "" $USERNAME

# Set password for the new SMB user
echo -e "$PASSWORD\n$PASSWORD" | sudo smbpasswd -a $USERNAME

# Configure Samba
echo "Configuring Samba..."
sudo mkdir -p $SHARED_DIRECTORY
sudo chmod $SHARED_DIRECTORY_PERMISSIONS $SHARED_DIRECTORY
sudo chown $SHARED_DIRECTORY_OWNER $SHARED_DIRECTORY

sudo bash -c 'cat >> /etc/samba/smb.conf <<EOF

[USERSHARE]
   comment = HOME
   path = '$SHARED_DIRECTORY'
   browseable = Yes
   writeable = Yes
   only guest = no
   create mask = 0777
   directory mask = 0777
   public = yes
   read only = no
   force user = root
   force group = root
   valid users = '$USERNAME'
EOF'

# Restart Samba service
echo "Restarting Samba service..."
sudo systemctl restart smbd

echo "Setup complete. User '$USERNAME' created with SMB share and static IP set to '$STATIC_IP'."
```
Save the file: `Ctrl+X` and `Y`<br>
Confirm it: `Y`

Now, will make the file executable and run the script to set the static IP:
```
chmod +x staticip.sh
sudo ./staticip.sh
```
![raspberry_11](https://github.com/user-attachments/assets/2ab60a82-3a43-4e9b-9fea-af2d8d507415)

Reboot the RPi. It would stop your PuTTY session. 
```
sudo reboot
```
It will stop the PuTTY connection, the IP is changed. 

![raspberry_12](https://github.com/user-attachments/assets/057b70a1-d5a1-4852-9724-827928337311)

After a few minutes, you can check your RPi new IP (step 2).

Then, you can re-connect by PuTTY (step 3) using the static IP you set: 192.168.1.113

## STEP 5: Check the storage disk is already mounted
Find your drive: 
```
lsblk
```
If you had used the script from step 3 a partition is already there: sda1.

![raspberry_13](https://github.com/user-attachments/assets/928e96c2-f717-487e-835a-02601e6c819c)

Formatting the disk:
```
sudo mkfs.ext4 /dev/sda1
```
If you had used the script from step 3 it will be already mounted. 

![raspberry_14](https://github.com/user-attachments/assets/da448682-c65d-4983-b4e5-5bc385072bb7)

It needs to ensure the “share” file is mounted every time the RPi reboots.

Access to edition mode to the fstab file, and fstab file will automatically mounts all filesystems at boot time (here you move with arrow keys):
```
sudo nano /etc/fstab
```
Add this line at the end: 
```
dev/sda1 /mnt/sda1/ ext4 defaults,noatime 0 1
```
![raspberry_15](https://github.com/user-attachments/assets/ecdc01cf-e0a3-45a0-8d46-b2692a0b8115)

Save the file: `Ctrl+X` and `Y`<br>
Confirm it: `Y`

Now you can reload the file "fstab" without to rebooting the system: 
```
sudo mount -av
```
![raspberry_16](https://github.com/user-attachments/assets/b8bf9ba0-0775-455e-91ce-b193cdac46b3)

## STEP 6: Create the folder you will share
Create a shared folder on the mount point of the partition we made in the storage drive and set the folder permissions:
```
sudo mkdir -p /mnt/sda1/shared
sudo chmod 0777 /mnt/sda1/shared
```
Set directory owner: *sudo chown pi /mnt/sda1/shared #sudo chown [your username created at step 1] /mnt/sda1/shared*<br> 
In this case, the username is "pi": 
```
sudo chown pi /mnt/sda1/shared #sudo chown pi /mnt/sda1/shared
```
Update and upgrade  the system again:
```
sudo apt update && sudo apt upgrade  -y
```
Check the drive is already mounted with right permissions:
```
ls -l /mnt
```
![raspberry_17](https://github.com/user-attachments/assets/c6cb6b63-eae1-4bec-99be-6bf2044e9dd4)

## STEP 7: Configure Samba software
Edit the information into smb.conf file:
```
sudo nano /etc/samba/smb.conf
```
At the end of the file copy and edit the next script:
```
[shared]
   path = /mnt/sda1/shared
   writeable = yes
   browseable = yes
   guest ok = no
   create mask = 0777
   directory mask = 0777
   read only = no
   valid users = pi #your user from step 1
   comment = Samba on Raspberry Pi
```
Save the file: `Ctrl+X` and `Y`<br>

Restart samba:
```
sudo systemctl restart smbd
```

## STEP 8: Maping Samba drive to access the network folder
### A) Map Samba drive in Windows 11
**Option 1: By the interface** 
Open File Manager<br>
In the menu, select `···`<br>
Select `Map network drive`<br>

![raspberry_18](https://github.com/user-attachments/assets/e43cc6df-a479-4034-99bb-b4267ed4420e)

Create the access to the network folder:
Select a not-used letter for the drive and set the folder address.
In this case, it would be: `Z:`
Folder: 
```
\\192.168.1.113\shared
```
And `Finish`
Remember, you had set the static IP and the name of the shared folder comes in the script.

![raspberry_20](https://github.com/user-attachments/assets/cfbc121d-805f-47b5-879b-6ac182e4f638)

**Option 2: By Command Prompt** 
Open the command prompt in your computer
```
net use Z: \\192.168.1.113\shared /user:pi
```
Remember, to set the information you set for your network.

### B): Map Samba drive in Ubuntu 22
In Terminal, update and upgrade the Ubuntu system:
```
sudo apt update && sudo apt upgrade -y
```
And install Samba client:
```
sudo apt install samba cifs-utils
```
Configure samba by editing the samba client file:
```
sudo nano /etc/samba/smb.conf
```
Add in the bottom of the file and edit it:
```
[shared]
path = /mnt/sda1/Shared
available = yes
valid users = pi # the one you set in your NAS
read only = no
browsable = yes
public  = yes
writeable = yes
```
Restart Samba:
```
sudo systemctl restart smbd
```
Create the mount point:
```
sudo mkdir -p /mnt/samba_share
```
Now, by the interface:
- Open Files and go to Other Locations:
- In the "Connect to Server" box, enter the IP address of your RPI, in this case:
```
smb://192.168.1.113/shared
```
Select `Connect`
Set username and password.

![raspberry_21](https://github.com/user-attachments/assets/0fcd4c91-f7c2-4986-89ab-6f278dcb16d0)

**Open the folder and start to share between your LAN devices!**      

![225be5b42cd3b82c3f93c5c274a028df (2)](https://github.com/user-attachments/assets/afe44107-e4ec-45ed-87d6-b5af568c90b2)

## Web references:
- https://www.raspberrypi.com/tutorials/nas-box-raspberry-pi-tutorial/
- https://www.raspberrypi.com/software/
- https://www.howtogeek.com/755213/how-to-map-a-network-drive-on-windows-11/















