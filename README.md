# openHAB Docker Installation Guide

Source: <https://github.com/jimtng/openhab-docker-setup>

This is a set of step-by-step copy-pasteable instructions to install openHAB as a Docker container, along with:

- Mosquitto MQTT Broker
- Zigbee2MQTT (optional)

This guide works on a clean, minimal install of Ubuntu 24, but it should also work on a working full installation and possibly also on other versions of Debian based Linux.

- [Expected Outcome](#expected-outcome)
- [Install Docker](#install-docker)
- [Create openhab user](#create-openhab-user)
- [Create Mosquitto Config](#create-mosquitto-config)
- [Create docker-compose.yaml](#create-docker-composeyaml)
- [Start up the Docker Containers](#start-up-the-docker-containers)
- [Your Next Steps](#your-next-steps)
  - [Configure Mosquitto](#configure-mosquitto)
  - [Configure Zigbee2MQTT](#configure-zigbee2mqtt)
- [More Tips](#more-tips)

## Expected Outcome

The steps in this guide will aim to create the following files and directories in your home directory:

- `docker-compose.yaml`
- `mosquitto/`
  - `mosquitto.conf`
- `zigbee2mqtt/`
  - `configuration.yaml`
- `openhab/`
  - `conf/`
  - `addons/`
  - `userdata/`

You can create a special user in your system to host/run these docker containers if you would like to separate them from your main user's home directory, but this isn't covered in this guide.

Users created on your system:

- `docker` created by docker installation
- `openhab:openhab` with a specific uid: 9001 and gid: 9001

Your current user will be added to the `docker` and `openhab` groups.

The commands in this guide will not overwrite your existing files, so if something isn't working, ensure that the file contents are correct.

By the end of this guide, you'll have docker up and running with a fresh installation of openHAB, Mosquitto and Zigbee2MQTT.
You'll need to finish configuring Zigbee2MQTT to connect your Zigbee dongle.

If you don't need/use Zigbee, edit docker-compose.yaml and remove the zigbee section.

## Install Docker

The following instructions were copied from <https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository>

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install the latest Docker version
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

Post install steps, modified from <https://docs.docker.com/engine/install/linux-postinstall/>

```shell
# Create the docker group if it does not exist
getent group docker || sudo groupadd docker
# This will add the current user to the docker group
sudo usermod -aG docker $USER
newgrp docker
```

## Create openhab user

```shell
sudo groupadd --gid 9001 openhab
sudo useradd -r -s /sbin/nologin --uid 9001 --gid 9001 openhab
sudo usermod -aG openhab $USER
```

You'll see a warning like this, it can be ignored:

> useradd warning: openhab's uid 9001 is greater than SYS_UID_MAX 999

## Create Mosquitto Config

```shell
cd
mkdir -p mosquitto
[ -f mosquitto/mosquitto.conf ] || cat <<-EOF > mosquitto/mosquitto.conf
per_listener_settings true

port 1883
protocol mqtt
allow_anonymous true
allow_zero_length_clientid true
connection_messages true    
EOF
```

## Create docker-compose.yaml

```shell
cd
[ -f docker-compose.yaml ] || cat <<EOF > docker-compose.yaml
services:
  openhab:
    image: openhab/openhab:latest
    container_name: openhab
    restart: always
    network_mode: host
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./.ssh:/openhab/.ssh
      - ./.karaf:/openhab/.karaf
      - ./conf:/openhab/conf
      - ./userdata:/openhab/userdata
      - ./addons:/openhab/addons
    environment:
      # Adjust accordingly
      OPENHAB_HTTP_PORT: 8080
      OPENHAB_HTTPS_PORT: 8443
      JAVA_MIN_MEM: 4g
      JAVA_MAX_MEM: 4g
  
  mosquitto:
    image: eclipse-mosquitto:latest
    container_name: mosquitto
    restart: always
    volumes:
      - ./mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf
    ports:
      # These ports should not be changed unless absolutely necessary
      - "1883:1883"
      - "8883:8883"

  zigbee2mqtt:
    image: koenkk/zigbee2mqtt:latest
    container_name: zigbee2mqtt
    restart: always
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./zigbee2mqtt:/app/data
    ports:
      # Adjust accordingly
      - "8088:8080"    
    # Uncomment the devices section below and map the correct device
    #devices:
      #- /dev/serial/by-id/usb-Silicon_Labs_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0:/dev/ttyUSB0
    depends_on:
      - mosquitto
EOF
```

## Start up the Docker Containers

```shell
docker compose up -d
```

## Your Next Steps

### Configure Mosquitto

At this point, Mosquitto will start and work just fine without requiring SSL certificate nor any authentications.

- You may want to adjust Mosquitto config (optional) to:
  - Set up username/password authentication
  - Optionally set SSL Certificate for mosquitto

### Configure Zigbee2MQTT

At this point Zigbee2MQTT isn't fuly configured.
You need to set up the dongle/serial port mapping.

Read up on how to configure Zigbee2MQTT from <https://www.zigbee2mqtt.io/guide/installation/02_docker.html>,
especially about configuring the serial port to communicate with yout zigbee dongle.

The Zigbee2MQTT config file is located in `zigbee2mqtt/configuration.yaml

## More Tips

- [Working with Docker, Upgrading/Downgrading](docker.md)
- [Working with MQTT / Mosquitto](mosquitto.md)
- [Backup/Restore](backup.md)
