## /data prefix:

By default all persistent data is saved under `/data`, you can override this path by setting the `DATA_FOLDER` env variable:

```sh
docker run -d --name vaultwarden \
  -e DATA_FOLDER=/persistent \
  -v /vw-data/:/persistent/ \
  -p 80:80 \
  vaultwarden/server:latest
```

Notice, that you need to adapt your volume mount accordingly.

## database name and location

Default is `$DATA_FOLDER/db.sqlite3`, you can change the path specifically for database using `DATABASE_URL` variable:

```sh
docker run -d --name vaultwarden \
  -e DATABASE_URL=/database/vaultwarden.sqlite3 \
  -v /vw-data/:/data/ \
  -v /vw-database/:/database/ \
  -p 80:80 \
  vaultwarden/server:latest
```

Note, that you need to remember to mount the volume for both database and other persistent data if they are different.

## attachments location

Default is `$DATA_FOLDER/attachments`, you can change the path using `ATTACHMENTS_FOLDER` variable:

```sh
docker run -d --name vaultwarden \
  -e ATTACHMENTS_FOLDER=/attachments \
  -v /vw-data/:/data/ \
  -v /vw-attachments/:/attachments/ \
  -p 80:80 \
  vaultwarden/server:latest
```

Note, that you need to remember to mount the volume for both attachments and other persistent data if they are different.

## icons cache

Default is `$DATA_FOLDER/icon_cache`, you can change the path using `ICON_CACHE_FOLDER` variable:

```sh
docker run -d --name vaultwarden \
  -e ICON_CACHE_FOLDER=/icon_cache \
  -v /vw-data/:/data/ \
  -v /icon_cache/ \
  -p 80:80 \
  vaultwarden/server:latest
```

Note, that in the above example we don't mount the volume locally, which means it won't be persisted during the upgrade unless you use intermediate data container using `--volumes-from`. This will impact performance as vaultwarden will have to re-download the icons on restart, but might save you from having stale icons in cache as they are not automatically cleaned.