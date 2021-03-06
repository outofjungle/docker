# Silent Protest Docker Configuration for Raspberry Pi

## Install Raspbian

Download [Rasbian Lite](https://www.raspberrypi.org/downloads/raspbian/).

Install to SD card:

```
sudo dd if=2017-04-10-raspbian-jessie-lite.img of=/dev/mmcblk0 bs=4M status=progress
sync
```

## Setup Raspbian

### Update Packages

```
sudo apt update
sudo apt upgrade
```

### Configure Rasbian

Run Raspbian config tool:

```
sudo raspi-config
```

* expand filesystem
* enable ssh
* set local
* set keyboard layout
* change password
* disable wait for network on boot

Install Tools:

```
sudo apt install git vim python-pip
```


### Setup Network

Edit network config to use both DHCP AND a static IP.

```
sudo vim /etc/network/interfaces
```

Edit to the following:

```
auto eth0
iface eth0 inet dhcp

auto eth0:0
iface eth0:0 inet static
address 192.168.1.6
netmask 255.255.255.0
```

With this configuration the Pi will get its default route and nameservers over DHCP, but will always have a static IP for us to use if we do not know its DHCP address.
Additionally this will allow us to distribute load over multiple Pis on the same network in the future.


#### Setup Hostnames

You should configure your network's DNS servers to return the IP address of the PI `192.168.1.6` for the domains `party.silentprotest.io`, `chat.party.silentprotest.io`, and `video.party.silentprotest.io`.

If you want to use another domain be sure to get the correct SSL certificates and update the Nginx configuration files.

## Reboot

`sudo reboot`


## Install Docker

Run Docker install script

```
curl -sSL https://get.docker.com/ | sh
```

Optionally, add the pi user to the docker group:

```
sudo usermod -aG docker pi
```

### Install Docker Compose

The default docker-compose install script does not support arm, so we will use pip.

```
sudo pip install docker-compose
```

## Setup Silent Protest Services

Clone this repository
```
git clone --depth 1 https://github.com/SilentProtest/docker.git silentprotest
cd silentprotest
```


Pull remote pre-built and base images

```
docker-compose pull
```

Build Custom images (optional). This may take a long time.
Building the images will also require you to clone this repo with the `--recursive` flag or run `git submodule update --init --recursive`.

```
docker-compose build
```


Generate HTTPS SSL certificates. [More information](www/README.md)


Clone [Party](https://github.com/silentprotest/party) UI Repo
```
git clone --depth 1 https://github.com/SilentProtest/party.git www/party
```

Start all services

```
docker-compose up -d
```

Optionally, watch the logs

```
docker-compose logs
```

### Reset: Nuke everything and start over

At any time you can run `reset.sh` to remove all docker containers volumes and networks.

# Party Time!

## Live Stream

Have your DJ stream to Icecast at `http://192.168.1.6:1337/dj` using the password configured inside `icecast.xml` If you change the password be sure to to change it in `radio.liq` too.

## Backup Files

If there is no live DJ, or you are waiting for one to connect, Silent Protest will automatically play media files in the `media/` directory of this repository. If you want to play off a USB flash drive change the mount point of `media` in the `docker-compose.yml` file.


