# ssbackup
Super Simple Backup is a bash wrapper around unix `tar`, which only does full or incremental backups. 
Backups will be `gzip`-ped; its parallel `pigz` counterpart is automatically used when available.

It is meant to be launched from `cron`, by creating simlinks like so:
```
/etc/cron.daily/backup_inc -> ~/bin/ssbackup*
/etc/cron.monthly/backup -> ~/bin/ssbackup*
```
Of course, your `ssbackup` may live somewhere else, and you may prefer different backup intervals. To run as user, use your user crontab entry.

Default is a full backup. There are three ways to initiate an incremental backup:
* launch through link named `backup_inc`, as shown above (in fact, if the link name contains `inc`)
* add parameter `inc` on the commandline
* add specific filename on the commandline (e.g. from the previous backup), which will be used as reference timepoint

For the first two cases, the `$target_dir` is searched for the latest succesful backup, which is then used as reference timepoint.

# Retries:
Most backup programs only try to do a backup once each interval (e.g., daily or montly), so that if it fails (because your usb drive wasn't plugged it), you miss one backup.

To enable multiple tries, we provide the `retry_backup` utility, so that you can e.g. try the daily backup each hour - success guaranteed ... eventually! Use it instead of the backup script itself, with simlinks like so:
```
/etc/cron.hourly/backup_inc -> ~/bin/retry_backup*
/etc/cron.montly/backup -> ~/bin/retry_backup*
```
Note, you'll need to set the interval in the config file (see below), or *also* have the ssbackup links as explained above (if you're running backups as user, your user crontab will be parsed to find corresponding backup entries). Options are hourly, daily, monthly, or yearly. The (lame) default is yearly.


# Customization:
Customization is done in the config file, where the following lines live:
```
# backup intervals; options are hourly, daily, monthly, yearly
# default is to get from /etc/cron.<interval> for root, or from user crontab
interval= # interval for full backup
interval_inc= # interval for incremental backup

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
If the `$target_dir` doesn't exist, or isn't writeable to the user, an error is printed and ssbackup terminates. (`/media/<user>/...` is where under Ubuntu external disks are mounted)

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
