# Raspberry-Server

### 1. Download Raspbian
   * [Raspbian](https://www.raspberrypi.org/downloads/raspberry-pi-os/)
   * [Etcher](https://etcher.io/)
   * Flash SD
  
### 2. Install all and configure the Raspberry
   * Change pasword user Pi
   * Activate [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/)
   * Add public key to [raspi](https://linuxhandbook.com/add-ssh-public-key-to-server/)
   * Add custom [user](https://www.lifewire.com/create-users-useradd-command-3572157) 

Modify users and passwd
```bash
sudo useradd username -G sudo
# copy this inside sudo visudo
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
# modify sshd_config for only access to your user
echo "AllowUsers username" | sudo tee -a /etc/ssh/sshd_config
# deny root access in ssh
echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config
# activate ssh
sudo systemctl enable ssh && sudo systemctl start ssh 
```

If wifi is activate then deactivate
```bash
sudo su
crontab -e
@reboot sudo ifconfig wlan0 down
```

Change IP and add static IP
```bash
# get current DNS
cat /etc/resolv.conf
# display your IP information
ip r | grep default
# edit DHCP  
sudo vim /etc/dhcpcd.conf
# change some values
interface eth0
static ip_address=<STATICIP>/24
static routers=<ROUTERIP>
static domain_name_servers=<DNSIP>
sudo reboot
```

### 3. Install packages

```bash
sudo apt-get update && sudo apt-get install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common \
     vim \
     fail2ban \
     ntfs-3g
```

### 4. Install GPG signatures and Docker repo

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

```bash
sudo apt-key fingerprint 0EBFCD88
```

### 5. Add Docker repo

```bash
echo "deb [arch=armhf] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```

### 6. Install Docker CE

```bash
sudo apt-get update && sudo apt-get install -y --no-install-recommends docker-ce docker-compose
```

### 7. Add your user to Docker group for avoid call Docker with sudo

```bash
sudo usermod -a -G docker pi
#(logout and login)
```

```bash
sudo vim /etc/default/docker
export DOCKER_TMPDIR="/mnt/storage/docker-tmp"
```

### 8. Mount NTFS disk

List all the devices.

```bashh
sudo su
fdisk -l
```
Select which one you need to mount.

```bash
sudo lsblk -f
ls -l /dev/disk/by-uuid/
```

Copy the UUID in `/etc/fstab`
```bash
# edit fstab
UUID="" /mnt/storage ntfs-3g defaults,auto 0 0
# mount disc
mount -a
sudo reboot
```

# Content

 * [Flex](https://github.com/jaymoulin/docker-plex)
 * [Flexget](https://github.com/Flexget/Flexget)
 * [Transmission](https://gitlab.com/jaymoulin/docker-transmission)

# Contributions

Repository based on :

* [pablokbs](https://github.com/pablokbs/plex-rpi)
* [IoTstack](https://github.com/SensorsIot/IOTstack)