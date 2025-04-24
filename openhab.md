# Working with openHAB

## Accessing openHAB

### Web UI Login

By default the port is 8080 unless you've changed it in `compose.yml`
If everything worked well, you should be able to access openHAB's web interface on <http://your.server.ip:8080/>

### Karaf Console

See <https://www.openhab.org/docs/administration/console.html>

## openHAB Log File

You can view openHAB's log file in the following directory:

```shell
tail -f ~/openhab/userdata/logs/openhab.log
```

## Clearing Cache

Sometimes openHAB can get into a strange state and it is helpful to clear its cache and temporary files so that it can re-initialize itself again.
This operation is perfectly safe, although it causes openHAB to take longer to start the first time after its cache is cleared.

Steps to clear openhab's cache:

```shell
docker compose stop
rm -rf ~/openhab/userdata/cache/* ~/openhab/userdata/tmp/*
docker compose start
```

## Running commands inside the container

This will bring you into the openhab's runtime environment, as root.

```shell
docker exec openhab bash
```

To become the `openhab` user inside the container:

```shell
su - openhab
```

Any command executed from your openhab scripts will be executed as the `openhab` user, so you can test it here.

Note that the docker compose.yml file maps the openhab user's `~/.ssh` directory to your real linux user's `~/.ssh` directory.
This means that openhab (scripts) will have access to any host you have access to using _your_ ssh private key.
If this is not desirable, you can map a special directory for openhab, e.g. `~/openhab/.ssh` in the compose.yml file.

You can also execute commands from inside the Karaf console.

[Back to main](README.md)
