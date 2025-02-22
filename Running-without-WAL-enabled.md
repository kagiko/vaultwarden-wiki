> [!IMPORTANT]
> WAL is an SQLite-specific setting. It is not applicable to MySQL or PostgreSQL; if you are using one of these backends, the `ENABLE_DB_WAL` config option has no effect.

By default during the startup `vaultwarden` will try to enable [WAL](https://sqlite.org/wal.html) for the DB. Adding this improves performance and in the past helped preventing failed requests under some circumstances.

## Reasons to turn WAL off

Generally speaking, unless you're absolutely sure that you need to turn WAL off, you should leave it enabled. However, there might be some valid cases to turn it off. For example:

* Some filesystems don't support WAL - this is especially true for network filesystems. If you're using such filesystem, the service will fail to start with `Failed to turn on WAL` error message.
* The database requires sqlite version `3.7.0` or newer, so if you for any reason (for example backups) require to access the DB directly with some other tool that cannot be updated, you might need to disable WAL.
* One of the [disadvantages described here](https://sqlite.org/wal.html#advantages) affects you

## How to turn WAL off

### 0. Make backup

These changes are generally safe and can be done without data loss, however [backing up your data](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault) prior to any changes is strongly advised.

### 1. Disable WAL on old DB

If you have old DB, that was used with WAL enabled, you need to disable it using sqlite:
1. Stop `vaultwarden`
2. Locate your [data folder](https://github.com/dani-garcia/vaultwarden/wiki/Changing-persistent-data-location). Normally there will be `db.sqlite3` file there unless you specified some other name to use.
3. Open the file using sqlite:

```bash
sqlite3 db.sqlite3
```
4. Disable WAL by typing `PRAGMA journal_mode=delete;` and pressing enter:

```
sqlite> PRAGMA journal_mode=delete;
delete
```
5. Quit sqlite utility by typing `.quit` and pressing enter. (notice the dot at the beginning)

### 2. Disable WAL in `vaultwarden`

To turn WAL off, you need to start `vaultwarden` with `ENABLE_DB_WAL` variable set to `false`:

```bash
docker run -d --name vaultwarden \
  -e ENABLE_DB_WAL=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
Make sure to always start with this variable present as starting even once without it will enable WAL again. (if that happens start at the [first step](#1-disable-wal-on-old-db) to disable it again)

## How to turn WAL on

Generally speaking you just start `vaultwarden` without `ENABLE_DB_WAL` set to false and server will automatically enable WAL for you. You can verify this by running:

```bash
sqlite3 db.sqlite3 'PRAGMA journal_mode'
```

Where `db.sqlite3` is the DB file used by `vaultwarden`. It should return back the mode that is currently used, so in our case it's `wal`. Disabled WAL will typically report back `delete` as this is the default.