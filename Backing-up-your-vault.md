## Overview

bitwarden_rs data should be backed up regularly, preferably via an automated process (e.g., cron job). Ideally, at least one copy should be stored remotely (e.g., cloud storage or a different computer). Avoid relying on filesystem or VM snapshots as a backup method, as these are more complex operations where more things can go wrong, and recovery in such cases can be difficult or impossible for the typical user. Adding an extra layer of encryption on your backups would generally be a good idea (especially if your backup also includes config data like your [[admin token|Enabling-admin-page]]), but you might choose to skip this step if you're confident that your master password (and those of your other users, if any) is strong.

## Backing up data

By default, bitwarden_rs stores all of its data under a directory called `data` (in the same directory as the `bitwarden_rs` executable). This location can be changed by setting the [[DATA_FOLDER|Changing-persistent-data-location]] environment variable. If you run bitwarden_rs with SQLite (this is the most common setup), then the SQL database is just a file in the data folder. If you run with MySQL or PostgreSQL, you will have to dump that data separately -- this is beyond the scope of this article, but a web search will turn up many other tutorials that cover this topic.

When running with the default SQLite backend, the bitwarden_rs `data` directory has this structure:

```
data
├── attachments          # Each attachment is stored as a separate file under this dir.
│   └── <uuid>
│       └── <random_id>
├── config.json          # Stores admin page config; only exists if the admin page has been enabled before.
├── db.sqlite3           # Main SQLite database file.
├── db.sqlite3-shm       # SQLite shared memory file (not always present).
├── db.sqlite3-wal       # SQLite write-ahead log file (not always present).
├── icon_cache           # Site icons (favicons) are cached under this dir.
│   ├── <domain>.png
│   ├── example.com.png
│   ├── example.net.png
│   └── example.org.png
├── rsa_key.der          # `rsa_key.*` files are used to sign authentication tokens.
├── rsa_key.pem
└── rsa_key.pub.der
```

When running with MySQL or PostgreSQL backends, the directory structure is the same, except there are no SQLite files. You'll still want to back up files in the `data` directory, as well as a dump of your MySQL or PostgreSQL tables.

Each set of files is discussed in more detail next.

### SQLite database files

_**Backup required.**_

The SQLite database file (`db.sqlite3`) stores almost all important bitwarden_rs data/state (database entries, user/org/device metadata, etc.), with the main exception being attachments, which are stored as separate files on the filesystem.

You should generally use the `.backup` command in the SQLite CLI (`sqlite3`) to back up the database file. This command uses the [Online Backup API](https://www.sqlite.org/backup.html), which SQLite documents as the [best way](https://www.sqlite.org/howtocorrupt.html#_backup_or_restore_while_a_transaction_is_active) to back up a database file that may be in active use. If you can ensure the database will not be in use when a backup runs, you can also use other methods such as the `.dump` command, or simply copying all the SQLite database files (including the `-shm` and `-wal` files, if present).

A basic backup command looks like this, assuming your data folder is `data` (the default):
```
sqlite3 data/db.sqlite3 ".backup '/path/to/backups/db-$(date '+%Y%m%d-%H%M').sqlite3'"
```
Assuming this command is run on January 1, 2021 at 12:34pm (local time), this backs up your SQLite database file to `/path/to/backups/db-20210101-1234.sqlite3`.

You can run this command via a cron job periodically (preferably at least once a day).

If you want to copy your backup data to cloud storage, [Rclone](https://rclone.org/) is a useful tool for interfacing with various cloud storage systems.

### The `attachments` folder

_**Backup required.**_

Attachments are the only important data not stored in database tables, mainly because they can be arbitrarily large, and SQL databases generally aren't designed to handle large blobs efficiently.

### The `rsa_key*` files

_**Backup recommended.**_

These files are used to sign the JWTs (authentication tokens) of users currently logged in. Deleting them would simply log out each user, forcing them to log in again.

### The `icon_cache` dir

_**Backup optional.**_

The icon cache stores [website icons](https://bitwarden.com/help/article/website-icons/) so that they don't need to be fetched from the login site repeatedly. It's probably not worth backing up unless you really want to avoid refetching a large cache of icons.

## Restoring backup data

Make sure bitwarden_rs is stopped, and then simply replace each file or directory in the `data` dir with its backed up version.

It's a good idea to run through the process of restoring from backup periodically, just to verify that your backups are working properly. When doing this, make sure to move or keep a copy of your original data in case your backups do not in fact work properly. 

## Examples

This section contains an index of third-party backup examples. You should review an example thoroughly and understand what it's doing before using it.

* https://github.com/shivpatel/bitwarden_rs-local-backup
* https://github.com/shivpatel/bitwarden_rs_dropbox_backup
* https://gitlab.com/1O/bitwarden_rs-backup
* https://github.com/jjlin/bitwardenrs-backup