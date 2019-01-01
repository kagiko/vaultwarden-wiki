## 1. the sqlite3 database

The sqlite3 database should be backed up using the proper sqlite3 backup command. This will ensure the database does not become corrupted if the backup happens during a database write.

```
mkdir $DATA_FOLDER/db-backup
sqlite3 /$DATA_FOLDER/db.sqlite3 ".backup '/$DATA_FOLDER/db-backup/backup.sqlite3'"
```

This command can be run via a CRON job everyday, however note that it will overwrite the same `backup.sqlite3` file each time. This backup file should therefore be saved via incremental backup either using a CRON job command that appends a timestamp or from another backup app such as Duplicati. To restore simply overwrite `db.sqlite3` with `backup.sqlite3` (while bitwarden_rs is stopped).

Running the above command requires sqlite3 to be installed on the docker host system. You can achieve the same result with a sqlite3 docker container using the following command.
```
docker run --rm --volumes-from=bitwarden bruceforce/bw_backup /backup.sh
```

You can also run a container with integrated cron daemon to automatically backup your database. See https://gitlab.com/1O/bitwarden_rs-backup for examples.

## 2. the attachments folder

By default, this is located in `$DATA_FOLDER/attachments`

## 3. the key files

This is optional, these are only used to store tokens of users currently logged in, deleting them would simply log each user out forcing them to log in again. By default, these are located in the `$DATA_FOLDER` (by default /data in the docker). There are 3 files: rsa_key.der, rsa_key.pem, rsa_key.pub.der.

## 4. Icon Cache

This is optional, the icon cache can re-download itself however if you have a large cache, it may take a long time. By default it is located in `$DATA_FOLDER/icon_cache`