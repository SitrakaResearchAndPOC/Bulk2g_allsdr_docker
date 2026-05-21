# Installing on PlutoSDR

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

# In Terminal 1
```
cpupower frequency-set -g performance 
```

# In the folder btspluto

```
docker build -t btspluto:v1 .
```
```
docker rm -f btspluto
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
  --name btspluto \
  --hostname btspluto \
  btspluto:v1
```
```
xhost +
```


# Testing driver
```
lsusb
```
Verify if this log exist </br>
`
Bus 001 Device 006: ID 0456:b673 Analog Devices, Inc. LibIIO based AD9363 Software Defined Radio [ADALM-PLUTO]  </br>
`
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
Unplug and replug PlutoSDR </br>
Find the IP address of PlutoSDR </br> </br>
```
ping 192.168.20.1
```
```
docker exec -ti btspluto bash -c 'ping 192.168.20.1'
```
```
docker exec -ti btspluto bash -c '$SRS_INSTALL/bin/SoapySDRUtil --info'
```
```
docker exec -ti btspluto bash -c '$SRS_INSTALL/bin/SoapySDRUtil --find'
```
```
docker exec -ti btspluto bash -c 'osmo-trx-soapy -C /opt/src/fork_osmo-trx_soapy/Transceiver52M/test1.cfg'
```
Tape ctrl+shift+T   </br>

launch service   :
```
docker exec -ti btspluto bash -c 'cd osmo-nitb-scripts && bash install_services.sh'
```
```
docker exec -ti btspluto python3 osmo-nitb-scripts/main_uhd_spoof.py
```


## Testing USRP SpoofScript1
```
docker exec -ti btspluto bash osmo-nitb-scripts/scripts_spoof1/finding_imsi_extenstion.sh```
```
You could find imsi and extension </br>
let's see for example IMSI as `646040222463674` and EXTENSION as `126` </br>
```
docker exec -ti btspluto  bash osmo-nitb-scripts/scripts_spoof1/set_imsi_extension.sh <IMSI> 0341220590
```
Verify by if the association is correct let's see for example imsi as 646040222463674 and extension as 0341220590
```
docker exec -ti btspluto bash osmo-nitb-scripts/scripts_spoof1/finding_imsi_extenstion.sh
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone </br>
Search GSM network (on your phone), associate with PLMN MCC `001` && MNC `01` </br>
Tape `*#001#` for finding your phone number (extension with osmo-bts) </br>
```
docker exec -ti btspluto  python2 osmo-nitb-scripts/scripts_spoof1/sending_sms_spoof_byextension.py
```
Sending for all extensions in osmo-bts
```
docker exec -ti btspluto  python2 osmo-nitb-scripts/scripts_spoof1/sending_sms_broadcast.py 
```
log should be : subscriber extension 0341220590 sms sender extension 0341220590 send ALERT Corona virus

## Testing USRP SpoofScript2
```
docker exec -ti btspluto python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
You could find imsi and extension Create a virtual extension `0341220590` and send sms to existing EXTENSION eg : `164`
```
docker exec -ti btspluto python2 osmo-nitb-scripts/scripts_spoof2/sms_send_source_dest_msg.py 0341220590 <EXTENSION> "link gmail"
```
You could find imsi and extension
```
docker exec -ti btspluto python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
docker exec -ti btspluto python2 osmo-nitb-scripts/scripts_spoof2/sms_spam.py <EXTENSION> 3 "link gmail"
```
You could find imsi and extension
```
docker exec -ti btspluto python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension `165`
```
docker exec -ti btspluto  python2 osmo-nitb-scripts/scripts_spoof2/sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension
```
docker exec -ti btspluto  python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py
```

## Testing USRP Fake SMS Sender
```
ping 192.168.20.1
```
```
docker exec -ti btspluto bash -c 'ping 192.168.20.1'
```
```
docker exec -ti btspluto bash -c '$SRS_INSTALL/bin/SoapySDRUtil --info'
```
```
docker exec -ti btspluto bash -c '$SRS_INSTALL/bin/SoapySDRUtil --find'
```
```
docker exec -ti btspluto bash -c 'osmo-trx-soapy -C /opt/src/fork_osmo-trx_soapy/Transceiver52M/test1.cfg'
```
Tape ctrl+shift+T   </br>

launch service   :
```
docker exec -ti btspluto bash -c 'cd osmo-nitb-scripts && bash install_services.sh'
```
```
docker exec -ti btspluto python3 osmo-nitb-scripts/main_uhd.py
```
Add victim phone and tape Tape ctrl+shift+T
```
docker exec -ti btspluto  cat osmo-nitb-scripts/interact.py
```
Change the the paramater default for add_argument in `/var/lib/osmocom/hlr.sqlite3`
```
docker exec -ti btspluto  python3 osmo-nitb-scripts/interact.py -c /config.json
```



