# 🚀 Raspberry-Server

### 🐧 1. Download Raspbian 

   * [Raspbian](https://www.raspberrypi.org/downloads/raspberry-pi-os/)
   * [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or [Etcher](https://etcher.io/)
   * Flash SD with the img (use lite for non graphical env)
  
### ⚙️ 2. Install all and configure the Raspberry

> __NOTE__ : Activate [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/) in the Raspberry

Create new user.
```bash
sudo useradd -m username
sudo passwd username
su username
```

Change **pi** and **root** password's.

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
# give sudo permissions to the created user
sudo usermod -aG sudo bis
# [Optional] edit sudo file for no password
sudo EDITOR=vim visudo
# [Optional] copy this inside 
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

```bash
# modify sshd_config for only access to your user
echo "AllowUsers username" | sudo tee -a /etc/ssh/sshd_config
# deny root access in ssh
echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config
# restart ssh
sudo systemctl restart ssh
# activate ssh if is not enable
sudo systemctl enable ssh && sudo systemctl start ssh 
```

**[Optional]** If wifi is activate then deactivate.

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

### 📦 3. Install packages

```bash
sudo apt-get update && sudo apt-get install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     git \
     software-properties-common \
     vim \
     fail2ban \
     ntfs-3g
```

### 🔐 4. Install GPG signatures and Docker repo

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

```bash
sudo apt-key fingerprint 0EBFCD88
```

### 🐋 5. Add Docker repo

Docker for armhf --> 32bit

```bash
echo "deb [arch=armhf] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```

Docker for arm64 --> 64bit

```bash
echo "deb [arch=arm64] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
```

### 🐋 6. Install Docker CE

```bash
sudo apt-get update && sudo apt-get install -y --no-install-recommends docker-ce docker-compose
```

### 👥 7. Add your user to Docker group for avoid call Docker with sudo

```bash
sudo usermod -aG docker $USER
#(logout and login)
```

[Optional] For storage docker tmp files in Disk.

```bash
sudo vim /etc/default/docker
export DOCKER_TMPDIR="/mnt/storage/docker-tmp"
```

### 💾 8. Mount NTFS disk

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
vim /etc/fstab
UUID="" /mnt/storage ntfs-3g defaults,auto 0 0
mkdir /mnt/storage
# mount disc
mount -a
sudo reboot
```

### 🚀 9. Start `rpi-server`

```bash
cd rpi-server
# Create Docker network Macvlan
# ip range: 192.168.1.193 - 192.168.1.254
docker network create -d macvlan --subnet 192.168.1.0/24 --ip-range 192.168.1.192/26 --gateway 192.168.1.1 -o parent=eth0 rpi-lan
# Configure .env file for custom values
vim .env
# Startup server
docker-compose up -d
```

## 🧪 Testing

- For testing Transmission with OpenVPN add this torrent from [TorGuard](https://torguard.net/checkmytorrentipaddress.php?hash=f1f5bda133bdbb4743773cc8548cbaee1fbff88a).

# 📚 Content

 * [Pihole](https://github.com/pi-hole/docker-pi-hole)
 * [Glances](https://github.com/nicolargo/glances)
 * [NextAlertX](https://github.com/jokob-sk/NetAlertX)
 * [Portainer](https://github.com/portainer/portainer)