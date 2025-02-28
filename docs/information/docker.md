---
---
# Docker
It is possible to run Zigbee2mqtt in a Docker container using the official [Zigbee2mqtt Docker image](https://hub.docker.com/r/koenkk/zigbee2mqtt/).

First run the container, this will create the configuration directory. Change `configuration.yaml` according to your situation and start again.

### Parameters
* `-v $(pwd)/data:/app/data`: Directory where Zigbee2mqtt stores it configuration
* `--device=/dev/ttyACM0`: Location of CC2531 USB sniffer

## Supported architectures
### amd64
```bash
docker run \
   -it \
   -v $(pwd)/data:/app/data \
   --device=/dev/ttyACM0 \
   -e TZ=Europe/Amsterdam \
   koenkk/zigbee2mqtt
```

### arm32v6 (E.G. Raspberry Pi)
```bash
docker run \
   -it \
   -v $(pwd)/data:/app/data \
   --device=/dev/ttyACM0 \
   -e TZ=Europe/Amsterdam \
   koenkk/zigbee2mqtt:arm32v6
```

### arm64v8
```bash
docker run \
   -it \
   -v $(pwd)/data:/app/data \
   --device=/dev/ttyACM0 \
   -e TZ=Europe/Amsterdam \
   koenkk/zigbee2mqtt:arm64v8
```

## Updating
To update to the latest Docker image:
```bash
docker rm -f [ZIGBEE2MQTT_CONTAINER_NAME]
docker rmi -f [ZIGBEE2MQTT_IMAGE_NAME] # e.g. koenkk/zigbee2mqtt:arm32v6
# Now run the container again, Docker will automatically pull the latest image.
```

## Tags
The following tags are available:
- Latest release version: `latest`, `arm32v6`, `arm64v8`
- Latest dev version (based on [`dev`](https://github.com/Koenkk/zigbee2mqtt/tree/dev) branch): `latest-dev`, `arm32v6-dev`, `arm64v8-dev`
- Locked to a specific release version: `X.X.X` (e.g. `0.2.0`), `X.X.X-arm32v6` (e.g. `0.2.0-arm32v6`), `X.X.X-arm64v8` (e.g. `0.2.0-arm64v8`)

__note__: since zigbee2mqtt images are manifest listed, there is no need to use explicit tags for the architecture. Docker will auto-detect the architecture and pull the right image.

## Support new devices
To add support for new devices, you'll need to git clone zigbee-shepherd-converters to ```$(pwd)/data/zigbee-shepherd-converters``` first:

```bash
git clone https://github.com/Koenkk/zigbee-shepherd-converters.git $(pwd)/data/zigbee-shepherd-converters
```

If you're integrating zigbee2mqtt with HomeAssistant, you'll also need to copy homeassistant.js (https://github.com/Koenkk/zigbee2mqtt/blob/master/lib/extension/homeassistant.js) to ```$(pwd)/data/lib/extension/homeassistant.js```.

Then run the docker command like this:

```bash
docker run \
   -it \
   -v $(pwd)/data:/app/data \
   -v $(pwd)/data/zigbee-shepherd-converters:/app/node_modules/zigbee-shepherd-converters \
   -v $(pwd)/data/lib/extension/homeassistant.js:/app/lib/extension/homeassistant.js \
   --device=/dev/ttyACM0 \
   -e TZ=Europe/Amsterdam \
   koenkk/zigbee2mqtt
```

After that follow the [guide](https://www.zigbee2mqtt.io/how_tos/how_to_support_new_devices.html) to add support for new devices.


## docker-compose Example
```yaml
  version: '3'
  services:
    zigbee2mqtt:
      container_name: zigbee2mqtt
      image: koenkk/zigbee2mqtt
      volumes:
        - ./data:/app/data
      devices:
        # CC251
        #- /dev/ttyUSB_cc2531:/dev/ttyACM0
        # CC2530 / GBAN GB2530S
        #- /dev/ttyUSB_cc2530:/dev/ttyACM0
      restart: always
      network_mode: host
      environment:
        - TZ=Europe/Amsterdam
```

## Docker Stack device mapping
Docker stack doesn't support device mappings with option `--devices` when deploying a stack in Swam mode. A workaround is to bind the device as volume binding and set the right permissions.

The workaround is based on the solution found at [Add support for devices with "service create"](https://github.com/docker/swarmkit/issues/1244#issuecomment-285935430), all credits goes this him.

1. Identify cc2531 device
	Identify the cc2531 device using the following command:

	```shell
	sudo lsusb -v
	```

	```
	Bus 001 Device 005: ID 0451:16a8 Texas Instruments, Inc.
	Device Descriptor:
	  bLength                18
	  bDescriptorType         1
	  bcdUSB               2.00
	  bDeviceClass            2 Communications
	  bDeviceSubClass         0 
	  bDeviceProtocol         0 
	  bMaxPacketSize0        32
	  idVendor           0x0451 Texas Instruments, Inc.
	  idProduct          0x16a8 
	  bcdDevice            0.09
	  iManufacturer           1 Texas Instruments
	  iProduct                2 TI CC2531 USB CDC
	  iSerial                 3 __0X00124B001936AC60
	  bNumConfigurations      1
	  Configuration Descriptor:
		bLength                 9
		bDescriptorType         2
		wTotalLength           67
		bNumInterfaces          2
		bConfigurationValue     1
		iConfiguration          0 
		bmAttributes         0x80
		  (Bus Powered)
		MaxPower               50mA
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        0
		  bAlternateSetting       0
		  bNumEndpoints           1
		  bInterfaceClass         2 Communications
		  bInterfaceSubClass      2 Abstract (modem)
		  bInterfaceProtocol      1 AT-commands (v.25ter)
		  iInterface              0 
		  CDC Header:
			bcdCDC               1.10
		  CDC ACM:
			bmCapabilities       0x02
			  line coding and serial state
		  CDC Union:
			bMasterInterface        0
			bSlaveInterface         1 
		  CDC Call Management:
			bmCapabilities       0x00
			bDataInterface          1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x82  EP 2 IN
			bmAttributes            3
			  Transfer Type            Interrupt
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0040  1x 64 bytes
			bInterval              64
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       0
		  bNumEndpoints           2
		  bInterfaceClass        10 CDC Data
		  bInterfaceSubClass      0 Unused
		  bInterfaceProtocol      0 
		  iInterface              0 
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x84  EP 4 IN
			bmAttributes            2
			  Transfer Type            Bulk
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0040  1x 64 bytes
			bInterval               0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x04  EP 4 OUT
			bmAttributes            2
			  Transfer Type            Bulk
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0040  1x 64 bytes
			bInterval               0
	Device Status:     0x0000
	  (Bus Powered)
	```

2. UDEV Rules

	Create a new udev rule for cc2531, `idVendor` and `idProduct` must be equal to values from `lsusb` command. The rule below creates device `/dev/cc2531`:

	```shell
	echo "SUBSYSTEM==\"tty\", ATTRS{idVendor}==\"0451\", ATTRS{idProduct}==\"16a8\", SYMLINK+=\"cc2531\",  RUN+=\"/usr/local/bin/docker-setup-cc2531.sh\"" | sudo tee /etc/udev/rules.d/99-cc2531.rules
	```

	Reload newly created rule using the following command:

	```shell
	sudo udevadm control --reload-rules
	```

3. Create docker-setup-cc2531.sh

	```shell
	sudo nano /usr/local/bin/docker-setup-cc2531.sh
	```

	Copy the following content:

	```shell
	#!/bin/bash
	USBDEV=`readlink -f /dev/cc2531`
	read minor major < <(stat -c '%T %t' $USBDEV)
	if [[ -z $minor || -z $major ]]; then
		echo 'Device not found'
		exit
	fi
	dminor=$((0x${minor}))
	dmajor=$((0x${major}))
	CID=`docker ps -a --no-trunc | grep koenkk/zigbee2mqtt | head -1 |  awk '{print $1}'`
	if [[ -z $CID ]]; then
		echo 'CID not found'
		exit
	fi
	echo 'Setting permissions'
	echo "c $dmajor:$dminor rwm" > /sys/fs/cgroup/devices/docker/$CID/devices.allow
	```

	Set permissions:

	```shell
	sudo chmod 744 /usr/local/bin/docker-setup-cc2531.sh
	```

4. Create docker-event-listener.sh

	```shell
	sudo nano /usr/local/bin/docker-event-listener.sh
	```

	Copy the following content:

	```shell
	#!/bin/bash
	docker events --filter 'event=start'| \
	while read line; do
		/usr/local/bin/docker-setup-cc2531.sh
	done
	```
	Set permissions:

	```shell
	sudo chmod 744 /usr/local/bin/docker-event-listener.sh
	```

5. Create docker-event-listener.service

	```shell
	sudo nano /etc/systemd/system/docker-event-listener.service
	```

	Copy the following content:

	```shell
	[Unit]
	Description=Docker Event Listener for TI CC2531 device
	After=network.target
	StartLimitIntervalSec=0
	[Service]
	Type=simple
	Restart=always
	RestartSec=1
	User=root
	ExecStart=/bin/bash /usr/local/bin/docker-event-listener.sh

	[Install]
	WantedBy=multi-user.target
	```

	Set permissions:

	```shell
	sudo chmod 744 /etc/systemd/system/docker-event-listener.service
	```

	Reload daemon

	```shell
	sudo systemctl daemon-reload
	```

	Start Docker event listener

	```shell
	sudo systemctl start docker-event-listener.service
	```

	Status Docker event listener

	```shell
	sudo systemctl status docker-event-listener.service
	```

	Enable Docker event listener

	```shell
	sudo systemctl enable docker-event-listener.service
	```

6. Verify and deploy Zigbee2Mqtt stack

	Now reconnect the cc2531. Verify using the following command:

	```shell
	ls -al /dev/cc2531
	```

	```shell
	lrwxrwxrwx 1 root root 7 Sep 28 21:14 /dev/cc2531 -> ttyACM0
	```

	Below an example of a `docker-stack-zigbee2mqtt.yml`:
	```yaml
	version: "3.7"
	services:
	  zigbee2mqtt:
		image: koenkk/zigbee2mqtt:latest-dev
		environment:
		  - TZ=Europe/Amsterdam
		volumes:
		  - /mnt/docker-cluster/zigbee2mqtt/data:/app/data
		  - /dev/cc2531:/dev/cc2531
		networks:
		  - proxy_traefik-net
		deploy:
		  placement:
			constraints: [node.hostname == rpi-3]
		  replicas: 1

	networks:
	  proxy_traefik-net:
		external: true
	```
	In the above example, `proxy_traefik-net` is the network to connect to the mqtt broker. The constraint makes sure Docker deploys only to this (`rpi-3`) node, where the cc2531 is connected to. The volume binding `/mnt/docker-cluster/zigbee2mqtt/data` is the zigbee2mqtt persistent directory, where `configuration.yaml` is saved.

	The zigbee2mqtt `configuration.yaml` should point to `/dev/cc2531`:

	```yaml
	[...]
	serial:
	  port: /dev/cc2531
	[...]
	```

	Deploy the stack:

	```shell
	docker stack deploy zigbee2mqtt --compose-file docker-stack-zigbee2mqtt.yml
	```
