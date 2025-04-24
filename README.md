# openHAB Docker Installation Guide

Source: <https://github.com/jimtng/openhab-docker-setup>

This is a set of step-by-step copy-pasteable instructions to install openHAB as a Docker container, along with:

- Mosquitto MQTT Broker
- Zigbee2MQTT (optional)

This guide works on a clean, minimal install of Ubuntu 24, but it should also work on a working full installation and possibly also on other versions of Debian based Linux.

Contents:

- [Expected Outcome](#expected-outcome)
- [Install Docker](#install-docker)
- [Create compose.yml](#create-composeyml)
- [Create Directories and Config Files](#create-directories-and-config-files)
- [Start up the Docker Containers](#start-up-the-docker-containers)
- [Access openHAB's Web Interface](#access-openhabs-web-interface)
- [Configure Zigbee2MQTT](#configure-zigbee2mqtt)
- [More Tips](#more-tips)

## Expected Outcome

- Docker and its dependencies installed
- 3 docker containers created
  - `openhab`
  - `mosquitto`
  - `zigbee2mqtt`
- The containers will run under your UID and GID. This is to make it easier to manage file permissions inside and outside the containers.
- Directories / Files created:
  - `compose.yml`
  - `openhab/`
    - `conf/` - the main openHAB's configuration folder, often referred to as `$OPENHAB_CONF`
    - `userdata/` - this is openHAB's internal storage for your instance, often referred to as `$OPENHAB_USERDATA`
    - `addons/` - this is where you can put custom add-on jar files to load into openHAB
  - `mosquitto/`
    - `mosquitto.conf`
  - `zigbee2mqtt/`
    - `configuration.yaml`
- Your unix user will be included in the `docker` group
- Zigbee2MQTT will not be fully configured yet. You'll need to finish configuring it to connect your Zigbee dongle.

The commands in this guide will not overwrite your existing files, so if something isn't working, ensure that the file contents are correct.

If you don't need/use Zigbee, edit `compose.yml` and remove the zigbee section.

## Install Docker

The following instructions were copied from <https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository>

```sh
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

Docker post install steps

```sh
# Create the docker group if it does not exist
getent group docker || sudo groupadd docker
# add the current user to the docker group
sudo usermod -aG docker $USER
# Load the new group
newgrp docker
```

Then do the next command to revert the main group back.

```sh
newgrp
[ $(id -u) == $(id -g) ] || echo "Warning: the current gid doesn't match your uid. Don't proceed. Try rebooting."
```

## Create compose.yml

```sh
GID=$(id -g)
[ -f compose.yml ] && echo Warning: compose.yml already exists || cat <<EOF > compose.yml
services:
  openhab:
    image: openhab/openhab:latest
    # image: openhab/openhab:milestone
    # image: openhab/openhab:snapshot
    container_name: openhab
    restart: always
    network_mode: host
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./.ssh:/openhab/.ssh
      - ./openhab/.karaf:/openhab/.karaf
      - ./openhab/conf:/openhab/conf
      - ./openhab/userdata:/openhab/userdata
      - ./openhab/addons:/openhab/addons
    environment:
      USER_ID: ${UID}
      GROUP_ID: ${GID}
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
    user: "${UID}:${GID}"
    ports:
      # These ports should not be changed unless absolutely necessary
      - "1883:1883"
      - "8883:8883"

  zigbee2mqtt:
    image: koenkk/zigbee2mqtt:latest
    container_name: zigbee2mqtt
    restart: always
    # Make it run as the openhab user too
    user: "${UID}:${GID}"
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
unset GID
[ -f docker-compose.yaml ] && echo Warning: docker-compose.yaml exists. compose.yml will take precedence.
```

## Create Directories and Config Files

```sh
mkdir -p openhab mosquitto zigbee2mqtt
[ -f mosquitto/mosquitto.conf ] || cat <<-EOF > mosquitto/mosquitto.conf
per_listener_settings true

port 1883
protocol mqtt
allow_anonymous true
allow_zero_length_clientid true
connection_messages true    
EOF
```

## Start up the Docker Containers

```sh
docker compose up -d
```

Check to see if the containers are running

```sh
docker compose ps
```

Set the mqtt host in Zigbee2mqtt config:

```sh
sed -i 's#mqtt://localhost:#mqtt://mosquitto:#' zigbee2mqtt/configuration.yaml
docker compose restart zigbee2mqtt
```

Congratulations! You've now got openHAB up and running!

## Access openHAB's Web Interface

You should now be able to access openHAB's web interface on <http://your.server.ip:8080/>

## Configure Zigbee2MQTT

At this point Zigbee2MQTT isn't fuly configured.
You need to set up the dongle/serial port mapping.

Read up on how to configure Zigbee2MQTT from <https://www.zigbee2mqtt.io/guide/installation/02_docker.html>,
especially about configuring the serial port to communicate with yout zigbee dongle.

The Zigbee2MQTT config file is located in `zigbee2mqtt/configuration.yaml`

## More Tips

- [Working with Docker, Upgrading/Downgrading](docker.md)
- [Working with openHAB](openhab.md)
- [Working with MQTT / Mosquitto](mosquitto.md)
- [Backup/Restore](backup.md)
- [How to delete docker images](docker-cleanup.md)
