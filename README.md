# ssbackup
Super Simple Backup is a wrapper around unix `tar`, which only does full or incremental backups. 
It is meant to be launched from `cron`, by creating simlinks like so:
```
/etc/cron.daily/backup_inc -> ~/bin/ssbackup*
/etc/cron.monthly/backup -> ~/bin/ssbackup*
```
Of course, your `ssbackup` may live somewhere else, and you may prefer different backup intervals.

Customization is performed in the header of the script, where the following lines live:
```
target_drive="/media/feenstra/Feenstra_VU_Back"
target_dir="$target_drive/Backupset2"

include_dirs="/ /home"
exclude_prune="/var /tmp"
exclude_dirs=".gvfs .thumbnails .trash .cache lock .tmp tmp .dropbox .git imapmail indexeddb"
exclude_files="windows\*.vdi"
exclude_paths="/var */recordings*/*.mp4 */Camera*/*.mp4"
```

`$include_dirs` are processed by `tar`, the `$exclude_*` by `find` which generates a list of dirs and/or files to be ignored by `tar`. Use the respective syntaxes! (You may need to read their manpages.)
In particular, the processing is done per white-space separated path as follows
* `$exclude_prune` are added with `-ipath <path> -prune`
* `$exclude_dirs` are added with `-iname <path> -prune`
* `$exclude_files` are added with `-iname <path>`
* `$exclude_paths` are added with `-ipath <path>`
Consequently, pathnames with embedded spaces are bound to break something. In particular, for the `$target_drive`/`_path` this will not work.

A succesful backup is marked by creating a file named `BACKUP_COMPLETE` in the directory. Conversely `BACKUP_FAIL` indicates somethign went wrong (most likely, your disk has filled up -- unless that also prevented this file from being created).

# Future plans:
Currently, only backups are made. Failed backups are not deleted. Importantly, there also is no cleanup policy to remove older backups. I'm planning to borrow the logic from [Simple Backup](https://launchpad.net/sbackup).
