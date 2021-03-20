# Raspberry-Server

### 1. Download Raspbian

   * [Raspbian](https://www.raspberrypi.org/downloads/raspberry-pi-os/)
   * [Etcher](https://etcher.io/)
   * Flash SD
  
### 2. Install all and configure the Raspberry

> __NOTE__ : Activate [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/) in the Raspberry

Create new user.
```bash
sudo useradd username
sudo passwd username
su username
```

**[Optional]** Change **pi** and **root** password's.

```bash
sudo passwd pi
sudo passwd root
```

Generate **SSH Key** on your computer to connect to the raspberry server.

```bash
ssh-keygen
cat ~/.ssh/id_rsa.pub
```

Copy ssh **public** key in `.ssh/authorized_keys` in the raspberry server.

```bash
ssh username@server_ip
# create .ssh folder if does not exists
mkdir -p .ssh && touch .ssh/authorized_keys
# copy public ip
vim .ssh/authorized_keys
# set the correct file permissions
chmod 700 .ssh && chmod 600 .ssh/authorized_keys
# change owner if its necessary
chown -R username:username .ssh
```

or install `ssh-copy-id`

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@server_ip
```

**[Optional]** Add more security by allowing only your user to access via ssh and denying root access via ssh.

```bash
# gice sudo permissions to the created user
sudo useradd username -G sudo
# [Optional] edit sudo file for no password
sudo visudo
# [Optional] copy this inside 
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
# modify sshd_config for only access to your user
echo "AllowUsers username" | sudo tee -a /etc/ssh/sshd_config
# deny root access in ssh
echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config
# activate ssh
sudo systemctl enable ssh && sudo systemctl start ssh 
```

If wifi is activate then deactivate.

```bash
sudo su
crontab -e
@reboot sudo ifconfig wlan0 down
```

**[Optional]** Change IP and add static IP.

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
# reboot system for apply changes
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

```bash
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

 * [Plex](https://github.com/jaymoulin/docker-plex)
 * [Flexget](https://github.com/Flexget/Flexget)
 * [Transmission](https://gitlab.com/jaymoulin/docker-transmission)
 * [Pihole](https://github.com/pi-hole/docker-pi-hole)
 * [Portainer](https://github.com/portainer/portainer)
 * [Samba](https://github.com/dperson/samba)

# Contributions

Repository based on :

* [pablokbs](https://github.com/pablokbs/plex-rpi)
* [IoTstack](https://github.com/SensorsIot/IOTstack)