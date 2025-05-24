# ssbackup
Super Simple Backup is a bash wrapper around unix `tar`, which only does full or incremental backups. 
Backups will be `gzip`ped; its parallel `pigz` counterpart is automatically used when available.

It is meant to be launched from `cron`, by creating simlinks like so:
```
/etc/cron.daily/backup_inc -> ~/bin/ssbackup*
/etc/cron.monthly/backup -> ~/bin/ssbackup*
```
Of course, your `ssbackup` may live somewhere else, and you may prefer different backup intervals.

Default is a full backup. There are three ways to initiate an incremental backup:
* launch through link named `backup_inc`, as shown above (in fact, if the link name contains `inc`)
* add parameter `inc` on the commandline
* add specific filename on the commandline, which will be used as reference timepoint

For the first two cases, the `$target_dir` is searched for the latest succesful backup, which is then used as reference timepoint.

Customization is done in the config file, where the following lines live:
```
target_host="user@hostname" # leave blank for local backup
target_drive="/media/<user>/<user>_Backup"
target_dir="$target_drive/Backupset2"
log_dir="/var/log/sbackup" # leave blank for no local (non external drive) log copies

include_dirs="/ /home"
exclude_prune="/var /tmp"
exclude_dirs=".gvfs .thumbnails .trash .cache lock .tmp tmp .dropbox .git imapmail indexeddb"
exclude_files="windows\*.vdi"
exclude_paths="/var */recordings*/*.mp4 */Camera*/*.mp4"
```
Drive and dir are assumed to exist, if they're not something will fail (not tested) (`/media/<user>/...` is where under Ubuntu external disks are mounted).

`$include_dirs` are processed by `tar`, the `$exclude_*` by `find` which generates a list of dirs and/or files to be ignored by `tar`. Use the respective syntaxes! (You may need to read their manpages.)
In particular, the processing is done per white-space separated path as follows
* `$exclude_prune` are added with `-ipath <path> -prune`
* `$exclude_dirs` are added with `-iname <path> -prune`
* `$exclude_files` are added with `-iname <path>`
* `$exclude_paths` are added with `-ipath <path>`

Consequently, pathnames with embedded spaces are bound to break something. In particular, for the `$target_drive`/`_path` this will not work.

A succesful backup is marked by creating a file named `BACKUP_COMPLETE` in the directory. Conversely `BACKUP_FAIL` indicates something went wrong (most likely, your disk has filled up -- unless that also prevented this file from being created).

# Future plans:
Currently, only backups are made. Failed backups are not deleted. Importantly, there also is no cleanup policy to remove older backups. I'm planning to borrow the logic from [Simple Backup](https://launchpad.net/sbackup).

See [src/sbackup/core/SnapshotManager.py](https://bazaar.launchpad.net/~sbackup-dev/sbackup/trunk/view/head:/src/sbackup/core/SnapshotManager.py)
```
        Logarithmic purge
        Keep progressivelly less backups into the past:
        Keep all backups from yesterday
        Keep one backup per day from last week.
        Keep one backup per week from last month.
        Keep one backup per month from last year.
        Keep one backup per quarter from 2nd last year.
        Keep one backup per year further in past.        
```
