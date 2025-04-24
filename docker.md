# Working with Docker

## Showing Container Status

```sh
docker compose ps
```

This command is similar to the unix `ps` command.

## Seeing container logs

If your container doesn't start up properly, e.g. keeps rebooting, you can see its log to find out what's wrong.

```sh
docker compose logs zigbee2mqtt
```

For more options:

```sh
docker compose logs --help
```

## Stopping/Starting a Container

```sh
docker compose [start|stop|restart] <containername>

# for example
docker compose stop openhab

# To stop all containers listed in your compose.yml file;
docker compose stop

# to restart
docker compose restart openhab
```

## Updating Container Version

Because in the example, all the images are set to `:latest`, you can upgrade them by issuing the following command:

```sh
# Upgrade openhab to the latest stable version
docker compose pull openhab
# After upgrading, reload must be done.
docker compose up -d openhab
```

The same can be done for mosquitto / other containers.

## Making Changes to compose.yml

If you modified the docker-compose.yaml file, run `docker compose up -d` to reload the changes. Simply restarting the docker container won't read the new changes.

## Downgrading To an Older / Specific Version

To downgrade openHAB to version 4.3.1 instead of the latest stable version, edit compose.yml, comment the `:latest` line and add:

```yaml
    # image: openhab/openhab:latest
    image: openhab/openhab:4.3.1
    # other options:
    # image: openhab/openhab:snapshot
    # image: openhab/openhab:milestone
```

Then do:

```sh
docker compose up -d openhab
```

To upgrade back to the latest (or any other) version, simply edit the `image:` line accordingly and run the `docker compose up -d openhab` again.

[Back to main](README.md)
