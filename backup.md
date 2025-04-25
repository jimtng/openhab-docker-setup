# Backup/Restore

To have a complete backup, include the following files/directories:

- `compose.yml`
- `openhab/` with some exclusions
- `mosquitto/mosquitto.conf`
- `zigbee2mqtt/configuration.yaml`

A backup can be performed while openhab is still running.

## Backup with Tar

### Create a tarball backup

```sh
DEST_DIR=~/backups
mkdir -p $DEST_DIR
cd
tar --verbose -czf $DEST_DIR/openhab_backup-$(date +'%Y-%m-%d').tgz \
--exclude=.git \
--exclude=openhab/conf/automation/ruby/.gem \
--exclude=openhab/userdata/backup \
--exclude=openhab/userdata/piper \
--exclude=openhab/userdata/cache \
--exclude=openhab/userdata/tmp \
compose.yml openhab mosquitto/mosquitto.conf zigbee2mqtt/configuration.yaml
unset DEST_DIR
```

Note you can create and send this tarball backup over ssh to another host too, e.g.:

```sh
tar -czf - .... | ssh user@host "cat > /path/to/target"
```

### Restore a tarball backup

Beware not to overwrite existing installation. Make a copy of `conf` and `userdata` folder first, if you're unsure.

```sh
tar -C $HOME -xf ~/backups/<tarballname>
```

## Rsync

### Create / update a backup copy

For example, we keep a backup copy in `~/backup`. Ideally this is located on a different disk, or over the network onto another machine.

```sh
cd
rsync -av --delete \
--exclude=.git \
--exclude=conf/automation/ruby/.gem \
--exclude=userdata/backup \
--exclude=userdata/piper \
--exclude=userdata/cache \
--exclude=userdata/tmp \
--include=mosquitto/mosquitto.conf \
--exclude=mosquitto/* \
--include=zigbee2mqtt/configuration.yaml \
--exclude=zigbee2mqtt/* \
compose.yml openhab mosquitto zigbee2mqtt ~/backup
```

[Back to main](README.md)
