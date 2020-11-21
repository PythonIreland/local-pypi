# local-pypi
Run a pypi server without internet connection


Idea is to have a Private WiFI network
and a raspberry pi to serve Python package,
so workshops at events can install dependencies even if venue's
network is bad or non-existent


install rasbian image to create SDCards
flash the lite version

set local network, keyboard layout, timezone, hostname ...

`sudo raspi-config`

wlan0

configured to connect at home, replace clear password with encrypted version.

encrypted version obtained with 
`wpa_passphrase celui_que_tu_veux`

/etc/wpa_supplicant/wpa_supplicant.conf

```
network={
        ssid="celui_que_tu_veux"
        psk=7d3f648eaa4b503a38f381423ea7b7c7f43e77b00961612e125236944c1f77dd
}

```

update system 

```
sudo apt update
sudo apt upgrade
sudo reboot now
```


## Install secondary WIFI driver

https://gist.github.com/sighmon/a5030b46e21c304c5697f31bb43dcc22


wget -O install-wifi.sh https://tinyurl.com/w7o2pum

sudo bash /home/pi/install-wifi.sh
sudo reboot now

you need to pass the full absolute path
It then installed the proper driver (8822bu)

/etc/hostapd/hostapd.conf


driver=8822bu
comment all lines starting with wpa


## Configure it as hotspot

https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md

http://nitlab.inf.uth.gr/mazi-guides/accessPoint.html

https://howtoraspberrypi.com/create-a-wi-fi-hotspot-in-less-than-10-minutes-with-pi-raspberry/

sudo systemctl disable raspapd.service





wlan1

AP, no password
dhcp up to 200 client

hostname = python-ireland-pypi

http://pypi.python.ireland

management ? ssh ?



sudo nano /etc/hosts
```
#ADD THE FOLLOWING LINES AT THE BOTTOM
10.0.0.1     local.mazizone.eu
10.0.0.1     portal.mazizone.eu
```



install docker

https://phoenixnap.com/kb/docker-on-raspberry-pi

https://github.com/X0Ken/docker-pypiserver-nginx


http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path

https://github.com/pypiserver/pypiserver/blob/master/docker-compose.yml



https://packaging.python.org/guides/index-mirrors-and-caches/#complete-mirror-with-bandersnatch

