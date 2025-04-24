# Working with Mosquitto / MQTT

## List the number of MQTT Clients connected to the broker

```sh
docker compose exec mosquitto mosquitto_sub -v -C 1 -t '$SYS/broker/clients/total'
```

## Monitor MQTT Messages

All zigbee2mqtt's traffic:

```sh
docker compose exec mosquitto mosquitto_sub -v -t 'zigbee2mqtt/#'
```

Or a specific device

```sh
docker compose exec mosquitto mosquitto_sub -v -t 'zigbee2mqtt/livingroom-light/#'
```

## Publish an MQTT Message

```sh
docker compose exec mosquitto mosquitto_pub -t 'some/topic/to/publish/to' -m 'data'
```

[Back to main](README.md)
