# local-pypi
Run a pypi proxy server without internet connection

Idea is to have a Private WiFI network and a raspberry pi to serve Python package,
so workshops at events can install dependencies even if venue's network is bad or non-existent

## Hardware

This is the hardware used, you can get it from Amazon (**Sponsored links**)
The exact setup is up to you 
- upgrade to Raspberry pi 4
- more or less disk space
- more or less powerful wifi antenna setup

What you need:

- A Raspberry pi, that is:
    - [Raspberry Pi starter Kit](https://amzn.to/3lZyRMo)

    or

    - [Raspberry Pi](https://amzn.to/360ZTxo)
    - [Enclosure for Pi](https://amzn.to/3fxacN0)
    - [External power for Pi](https://amzn.to/2UVX5LD)
    - [Small microSD card](https://amzn.to/33f9TBl)

- [Heat sink](https://amzn.to/3fAv99F) or [fan](https://amzn.to/3l2FWdT) for Pi 
- [USB dongle with antenna](https://amzn.to/3m4uwHR)
- [Powered USB hub](https://amzn.to/3ftgbCk)
- [External power for USB hub](https://amzn.to/3604NLd)
- [Large external UDB3 HDD](https://amzn.to/2J3ZZeP)

keyboard and screen might make your life easier.
It is important you power the USB hub from an external source.
Please, let us know if there is broken link.
You're looking at 160 € give or take.


## Configure External HDD

- Format it in ext4
- Give it a [nice label](https://askubuntu.com/questions/194510/how-to-edit-label-of-usb-drive) like `PypiMirror`

## Configure raspberry

install [rasbian image](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) to create SDCards
flash the lite version

set local network, keyboard layout, timezone, hostname ...

`sudo raspi-config`

Set hostname to `python-ireland-pypi`

### Attach to local WiFi
configured to connect at home, replace clear password with encrypted version.

encrypted version obtained with `wpa_passphrase <ssid>`

then replace clear password in `/etc/wpa_supplicant/wpa_supplicant.conf`

```
network={
        ssid="celui_que_tu_veux"
        psk=7d3f648eaa4b503a38f381423ea7b7c7f43e77b00961612e125236944c1f77dd
}
```

https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md


### Update system 

```
sudo apt update
sudo apt upgrade
sudo apt install openssh-server
sudo reboot now
```

### Install secondary WIFI driver

Download the [install-wifi.sh](https://gist.github.com/sighmon/a5030b46e21c304c5697f31bb43dcc22) script.

wget -O install-wifi.sh https://tinyurl.com/w7o2pum


`wget -O install-wifi.sh https://gist.github.com/sighmon/a5030b46e21c304c5697f31bb43dcc22`

Then run it. you need to pass the full absolute path.
It detects and installs the proper driver (8822bu in this instance), take note of driver name

```
sudo bash /home/pi/install-wifi.sh
sudo reboot now
```

## Configure it as hotspot

### install RaspAP

https://github.com/billz/raspap-webgui

Follow procedure, say
- no to ad blocking

- In Authentication
    - change admin password
- In Hotspot...Basic settings
    - change SSID to `python-ireland-pypi`
    - change Access Point to wlan1 (dongle with powered antenna)
- In System...Advanced
    - set Web server port to 90 (leaving 80 for pypi nginx)


### Polishing

- Remove clear password from `/etc/wpa_supplicant/wpa_supplicant.conf`
- Check listen port in `/etc/lighttpd/lighttpd.conf` is set to 90

Access Point admin will be up at http://10.3.141.1:90/

`sudo nano /etc/hosts`
```
#ADD THE FOLLOWING LINES AT THE BOTTOM
10.3.141.1     pypi.python.ireland
10.3.141.1     pypi.python.ie
10.3.141.1     pypi.pyie
10.3.141.1     pypi.python-ireland
```
so as to make admin available at http://pypi.pyie:90/

## mount disk

https://www.raspberrypi.org/documentation/configuration/external-storage.md

```
sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL
UUID                                 NAME        FSTYPE  SIZE MOUNTPOINT           LABEL      MODEL
                                     sda                 1.8T                                 External_USB_3.0
a2a475b9-cc28-470e-8561-020126797fa9 `-sda1      ext4    1.8T /media/pi/PypiMirror PypiMirror 

```

```
sudo mkdir -p /media/pi/PypiMirror
sudo mount /dev/sda1 /media/pi/PypiMirror

pi@python-ireland-pypi:~ $ sudo blkid
...
/dev/sda1: LABEL="PypiMirror" UUID="a2a475b9-cc28-470e-8561-020126797fa9" TYPE="ext4" PARTLABEL="Pypi" PARTUUID="52c515af-b07c-46a9-9795-1f82b7ac105a"
...

pi@python-ireland-pypi:~ $ sudo nano /etc/fstab

```


add line like
```
PARTUUID=52c515af-b07c-46a9-9795-1f82b7ac105a  /media/pi/PypiMirror               ext4    defaults,auto,users,rw,nofail,x-systemd.device-timeout=30 0 0
```
at EOF


## install docker

follow the procedure :
https://phoenixnap.com/kb/docker-on-raspberry-pi

```
sudo apt update
sudo apt install libxslt1-dev git screen docker-compose
```

### build bandersnatch image for Raspbian lite arm

```
git clone https://github.com/pypa/bandersnatch
cp Dockerfile.pi ./bandersnatch
cd bandersnatch
docker build --no-cache -t pyie/bandersnatch:pi -f Dockerfile.pi .
```

### Install configuration (This project)

```
git clone https://github.com/PythonIreland/local-pypi.git
```


### Run stack

```bash
cd local-pypi
screen
sudo docker-compose up
```
You can then detach with `Ctrl+A D`


#### update repo

if new requirements file in bandersnatch config, or new packaged required within one of the file

```
# enter the bandersnatch container
sudo docker exec -it pypi-local_bandersnatch_1 bash
# update new packages
bandersnatch mirror --force-check
```

make sure you are connected through `wlan0` or `eth0`

### Checkout configuration
http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path
https://github.com/pypiserver/pypiserver/blob/master/docker-compose.yml

check external USB Disk mount point (/media/pi/PyPiMirror)

https://packaging.python.org/guides/index-mirrors-and-caches/#complete-mirror-with-bandersnatch


## Set Clients

- you're on the `python-ireland-pypi` network

```
pip install --index-url=http://pypi.pyie/simple --trusted-host pypi.pyie requests
```


## Troubleshooting

no screen at boot for Lite image:
https://www.raspberrypi.org/forums/viewtopic.php?t=34061

Best to simply login through ssh (VGA is old fashioned)

make sure the external disk is mounted, otherwise process will
fill up completely the SD Card.


## links

lower level setupi
https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md

fan control
https://howchoo.com/g/ote2mjkzzta/control-raspberry-pi-fan-temperature-python

pypi server 
https://github.com/X0Ken/docker-pypiserver-nginx
