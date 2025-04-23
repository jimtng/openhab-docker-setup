# Backup/Restore

To back up your openhab, simply backup the `~/openhab/conf` and `~/openhab/userdata` using whatever means you prefer. You don't need to stop openhab prior to taking a backup.

Tip: exclude `~/openhab/userdata/backup` from your backup.

Examples:

## Backup with Tar

### Create a tarball backup

```shell
DEST_DIR=~/backups
mkdir -p $DEST_DIR
cd ~/openhab
tar --verbose -czf $DEST_DIR/openhab_backup-$(date +'%Y-%m-%d').tgz \
--exclude=.git \
--exclude=conf/automation/ruby/.gem \
--exclude=userdata/backup \
--exclude=userdata/piper \
--exclude=userdata/cache \
--exclude=userdata/tmp \
conf userdata
unset DEST_DIR
```

Note you can create and send this tarball backup over ssh to another host too, e.g.:

```shell
tar -czf - .... | ssh user@host "cat > /path/to/target"
```

### Restore a tarball backup

Beware not to overwrite existing installation. Make a copy of `conf` and `userdata` folder first, if you're unsure.

```shell
cd ~/openhab
tar -xf ~/backups/<tarballname>
```

## Rsync

### Create / update a backup copy

For example, we keep a backup copy in `~/backup`. Ideally this is located on a different disk, or over the network onto another machine.

```shell
DEST_DIR=~/backup
mkdir -p $DEST_DIR
cd ~/openhab
rsync -av --delete \
--exclude=.git \
--exclude=conf/automation/ruby/.gem \
--exclude=userdata/backup \
--exclude=userdata/piper \
--exclude=userdata/cache \
--exclude=userdata/tmp \
conf userdata $DEST_DIR
unset DEST_DIR
```

[Back to main](README.md)
