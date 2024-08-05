# **How to build a Raspberry Pi NAS**
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

## **STEP 1: Prepare the Raspberry PI image**
Download the RPI OS image software [here](https://www.raspberrypi.com/software/).
I use **Imager 1.8.5** download for Windows.
<br>
Select the settings:
- Raspberry Pi Device: **RASPBERRY PI 3**
- Operating system: **RASPBERRY PI OS 64BIT**
- Storage: **SDHC CARD**
<br>
Adjust the settings: 
- Set username and password of your system (in this case it is pi / pi)
- Add wifi information.
- Enable ssh
<br>
Open it and ignore pop-up messages that can come out. 
<br>
Once it is completed with the writing and verifying, set the SDHC card into the RPI, power it on and connect the hard disk storage and the periferics elements (keyboard, screen and mouse). 
I also I also connected my RPI to the internet by Ethernet.
<br>
## **STEP 2: Find the IP of the RPI**
Options:
1. Via router DHCP leases, look for the RPI system name, you gave the system in the image you installed in the SDHC card.
2. Via router by the MAC address in the list of connected devices on the router settings.
3. c.	Open the terminal and write the command: 
```
ifconfig
```
<br>
## **STEP 3: Acces to your RPI in remot**
<br> 
When you connect remotely by PuTTY you will not need the peripherals anymore.
Open PuTTy.
Host name: the IP of your RPI.
The prompt will open and will ask you for user and password. 
In this case: pi / pi
<br>
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
<br>
## **STEP 4: Configure the storage**
Find your drive: 
```
lsblk
```
If you had used the script from the step 3 a partition is already there: sda1.



