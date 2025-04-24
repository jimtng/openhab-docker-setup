# How to remove docker containers and images

## Remove docker containers

```shell
docker compose stop
# Remove ALL containers in compose.yml
docker compose rm
```

## Remove docker images


List the images

```shell
docker compose image ls | egrep 'zigbee2mqtt|openhab|mosquitto'
```

If you are sure those images are the ones to be removed:

```shell
# Remove them
docker image ls | egrep 'zigbee2mqtt|openhab|mosquitto' | awk '{print $3}' | xargs docker image rm
```
