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
In particular, the processing is done as follows
* $exclude_prune 
