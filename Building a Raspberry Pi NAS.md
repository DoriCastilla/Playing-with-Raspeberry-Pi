# **How to build a Raspberry Pi NAS. Guide for beginners**
Objective: to install and configure a Raspeberry Pi NAS in a SOHO LAN
Components: 
- Raspeberri Pi 3b+ (RPI from now)
- SDHC card 16GB
- Hard disk or USB for starage disk
- RPI image 
- PuTTy software (optional)
- Ehternet (optional)
- Screen   
- Keyboard
- Mouse

## STEP 1: Prepare the Raspberry PI image
Download the RPI OS image software [here](https://www.raspberrypi.com/software/).
I use **Imager 1.8.5** download for Windows.
 
Select the settings:
- Raspberry Pi Device: **RASPBERRY PI 3**
- Operating system: **RASPBERRY PI OS 64BIT**
- Storage: **SDHC CARD**
 
Adjust the settings: 
- Set username and password of your system (in this case it is pi / pi)
- Add wifi information.
- Enable ssh
 
Open it and ignore pop-up messages that can come out. 
 
Once it is completed with the writing and verifying, set the SDHC card into the RPI, power it on and connect the hard disk storage and the periferics elements (keyboard, screen and mouse). 
I also I also connected my RPI to the internet by Ethernet.
 
## STEP 2: Find the IP of the RPI
Options:
1. Via router DHCP leases, look for the RPI system name, you gave the system in the image you installed in the SDHC card.
2. Via router by the MAC address in the list of connected devices on the router settings.
3. c.	Open the terminal and write the command: 
```
ifconfig
```
 
## STEP 3: Access to your RPI in remot
<br> 
When you connect remotely by PuTTY you will not need the peripherals anymore.
Open PuTTy.
Hostname: the IP of your RPI.
The prompt will open and will ask you for user and password. 
In this case: pi / pi
 
Update and upgrade the system: 
```
sudo apt update && sudo apt upgrade -y
```
Set the static IP and install and config Samba in the staticip.sh : 
```
nano staticip.sh
```
And write down this script and edit it with your own information where needed: 
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
```
Continue the script to set the static IP:
```
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
```
This command will install Samba:
```
# Update and install samba
echo "Updating system and installing Samba..."
sudo apt-get update
sudo apt-get install samba samba-common-bin -y
```
This will create the user, with the username and password you did set: 
```
 Create a new user
echo "Creating new user..."
sudo adduser --disabled-password --gecos "" $USERNAME

# Set password for the new SMB user
echo -e "$PASSWORD\n$PASSWORD" | sudo smbpasswd -a $USERNAME
```
Configure and restart Samba:
```
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
Save the file: Ctrl+X and Yes.
Make the file executable:
```
chmod +x staticip.sh
```
Run the script:
```
sudo ./staticip.sh
```
Reboot the RPI. It would stop your PuTTY session. After few minutes you can re-connect by PuTTY setting the static IP you set.
 
## STEP 4: Configure the storage
Find your drive: 
```
lsblk
```
If you had used the script from step 3 a partition is already there: sda1.

![Screenshot 2024-08-02 151915](https://github.com/user-attachments/assets/c6fd37ce-b1ca-45b6-889a-d2ecf44cfa9f)

If the partition is not there: 
```
sudo fdisk /dev/sda
```
## STEP 5: Prepare mount point and mount disk
This is the command for formatting the disk:
```
sudo mkfs.ext4 /dev/sda1
```
If you had used the script from step 3 it will be already mounted. Go to next step.

![Screenshot 2024-08-02 152445](https://github.com/user-attachments/assets/10ba8987-36ea-49f7-8cfd-a01b651a7d8b)

If not mounted:

![Screenshot 2024-07-08 212705](https://github.com/user-attachments/assets/3bc50103-d5c1-4a36-b79c-08876d9150c8)

You must create a mount point:
```
sudo mkdir -p /mnt/sda1
```
Ensure the “share” file is mounted every time the RPI reboots. Access to Edition mode to the fstab file, and fstab file will automatically mounts all filesystems at boot time:
```
sudo nano /etc/fstab
```
Add this line at the end: 
```
dev/sda1 /mnt/sda1/ ext4 defaults,noatime 0 1
```
![Screenshot 2024-07-08 215338](https://github.com/user-attachments/assets/920bb773-fc0b-413a-a2d0-ca52adb9ecb0)

Ctrl+x and Y to save the changes.
Now you must reload the file "fstab", to do it without to rebooting the system: 
```
sudo mount -av
```
Check the drive is already mounted:
```
ls -l /mnt
```
![Screenshot 2024-08-02 112132](https://github.com/user-attachments/assets/7a20d97a-6446-4844-bd17-dc235b9c9ee3)

## STEP 6: Create the folder you will share
Create a shared folder on the mount point of the partition we made in the storage drive:
```
sudo mkdir -p /mnt/sda1/shared
```
Set the folder permissions:
```
sudo chmod 0777 /mnt/sda1/shared
```
Set directory owner. In this case, the username is "pi": 
```
sudo chown pi /mnt/sda1/shared #sudo chown [your username created at step 5] /mnt/sda1/shared
```
Install Samba software. If you used the script from Step 3, it is be already installed.
If not, use this command:
```
sudo apt install -y samba samba-common-bin
```
Update and upgrade  the system again:
```
sudo apt update && sudo apt upgrade  -y
```
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
   valid users = pi #your user from step 3
   comment = Samba on Raspberry Pi
```
Restart samba:
```
sudo systemctl restart smbd
```
## STEP 8: Map Samba drive to access the network folder
### STEP 8.1: Map Samba drive in Windows 11
Option 1: 
By the interface:

![Screenshot 2024-07-08 221141](https://github.com/user-attachments/assets/0f7f9244-c598-4c4a-b17c-9a225068be45)

Create the access to the network folder:
Select a not-used letter for the drive and set the folder address.
In this case, it would be: 
Z:
\\192.168.1.113\shared 
Remember, you had set the static IP and the name of the shared folder comes in the script.

![Screenshot 2024-07-09 141955](https://github.com/user-attachments/assets/81004763-fba9-460a-9ba9-7d221d4bcc19)

Option 2: 
By Command Prompt:
```
C:\Users\quedi>net use Z: \\192.168.1.113\shared /user:pi
```
Remember to set the information of your network.
### STEP 8.2: Map Samba drive in Ubuntu 22
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
Click Connet.
Set username and password.

Open the folder and start to share between your LAN devices!








