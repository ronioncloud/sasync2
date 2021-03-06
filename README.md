## **SASYNC 4.1 ~= SASYNX 1.1**
- [NOTE] `sasync` works as usual, with messages, notifications, etc. 

- [NEW] Introducing `sasynx`, rewrite of sasync with ~90% reduction in lines of code
  - `sasynx` is more simple but also more flexible than `sasync`
  - `sasynx` retains many features of sasync, while adding tree-sync, chain-sync and group syncs
  - Before running `sasynx` copy sasynx.conf-sample to sasynx.conf and edit as needed
- The only files required are `sasynx`, `sasynx.conf` and a set file e.g. `sasynx.set`
- The sasynx.config file uses many of the same variables as the old sasync.conf
- Supports multiple sets in a single line 
  - `./sasynx tv movies` will run both the tv and movies sets
- `sasynx` depends on fclone, lclone or gclone to auto rotate service accounts
  - choose your *clone in the config file or via the -r flag
  - `./sasynx -r fclone tv` will use fclone
- Default action `sync`, `copy` `move` can be set in config file or via `-a flag`
  - `./sasynx -a copy tv` will copy files
- Dedupe both source and destination folders via `-d` flag
- Send remote stats to gsheet via `-z` flag
- Default sync from source to single or multiple destinations (a `tree` sync)
- Can be changed to a `chain` sync with `-c` flag. This will copy source to dest1, then dest1 to dest2 and so on
- Sets can be literal or multi-layered
  - Literal, single layered e.g. 
    ```
      src=("td1:movies")
      dest=("td2:movies")```
  - Multi-layered e.g.
      ```
    movies)
        src=("td1:")
        src_dir=("movies")
        dest=("td2:" "td3:")
        dest_dir=("Media/movies")
      ```
- Multiple source-dest sets can be grouped together
      ```
      media) group=("tv" "movies")
      ```
  - `./sasynx media` will run both the tv and movies sets
- All commands can be tested without running using `-e` flag
- OPTIONAL: As with sasync, `sizeup` feature requires some python dependencies

## **SASYNC 3.5 **
Ref: https://github.com/88lex/sa-guide For more info

###  Changelog V3.5
- [NEW] Added -f flag to run sync/copy with mawaya's fclone.
- [NEW] Added -l flag to run sync/copy with l3uddz's lclone.
- [NEW] Added -g flag to run sync/copy with donwa's gclone.
- [NEW] Added sizeup to push folder size and file count to gsheet (not yet documented)


SASYNC uses rclone and Google Service Accounts (SAs) to sync, copy or move files between rclone remotes.

#### <br>Usage: &emsp; `./sasync set.file ` &emsp; &emsp; 
- `./sasync -t set.file` to run sasync inside tmux
- `./sasync -p 3 set.file` to run sasync sets in 3 parallel tmux windows. `3` can be any number
- `./sasync -c my.conf set.file` to run sasync with your custom config file
- `./sasync set.file1 set.file2` to run multiple sets in sasync.
- Above flags/options may be combined.
- Enable sasync execution with `chmod +x sasync`

### **How does it work?**
`sasync` reads from a **set file**. Each line in the set file describes 
- An action (sync, copy, move) that rclone will run 
- A source and destination
- Any additional rclone flags which you would like to apply to source/destination pairs
<pre>
sync      teamdrive:docs     my_td:docs     --max-age=3d
copy      teamdrive:photos   my_td:photos
move      teamdrive:misc     my_td:misc     --min-size=1M
</pre>

- Spaces or tabs as field separators are the easiest for sasync to parse.
- If you have remote or folder names with spaces then you MUST use , or | as field separators. You need to change IFS1 in `sasync.conf` if using `,` or `|` .

<br>

###  Destination requirements

- Service Accounts (SAs) can be used to write to a Shared/Team Drive. Otherwise sasync will run, but will not use SAs and quotas may be limited by the single account.
- read/write/delete (RWD) permissions are necessary

###  Source requirements

- Must have read permission
- Best case if the source is a TD
- If source is not a TD then you MUST use `CHECK_REMOTES=true` in the config file. Otherwise sasync will not work
- If all of the above are true then SAs will rotate with both source and destination, and sync/copy limits can be higher. Each SA has a 750GB upload/inbound quota and a 10TB download/outbound quota
- If you see permission/access errors, it is a good idea to run `checkremotes` in /sasync/utils. Also see the following guide to manually checking permissions. https://github.com/88lex/sa-guide/blob/master/rclone_remote_permissions.md


### Optional Features toggled in sasync.conf
- Add `-p N` to run sasync in tmux with N parallel instances

- Add `-c file.conf` before the set file to use a different config file

- Add `-t` to force a single instance of sasync to run in a tmux window

- Add rcloneflag(s) to the command line after the set file

- Check if each source and destination remote exists in the rclone.conf file and if they are Team Drives (now called Shared Drives)

- Check if each remote has read and service account permissions

- Clean Source and Destination remotes of duplicates and trash. Can be done before and/or after the sasync runs rclone copy/sync/move

- Check if file count in Source and Destination are equal

  - If equal then skip to the next source-destination pair in the set file

- Check if total file size in Source and Destination are equal

  - If equal then skip to the next source-destination pair in the set file

  - If not equal then estimate number of SA's required

- Choose whether to EXIT or CONTINUE when there is a problem with any source-destination pair in a set file

- The `sasync.conf` file is backed up daily to a `./backup` folder and kept for 14 days (default, adjustable)

- The config file in the repo is called **`sasync.conf.default`** so you do not not overwrite an existing `sasync.config`

- The default `filter` file excludes some system files. Open and customize it to suit your own needs

- Flexible unlimited rclone flags in the set files:  Add multiple rclone flags to each/all pairs in the set.* file. Flags in the set file will override flags in the config file

- Log files sasync creates two log files with each run. `stderr_set.name.log` and `stdout_set.name.log` and puts them in the `logs` folder

<br>

### Notes for modifying the sasync.conf file

- The template config file in the repo is called sasync.conf.default. Before running sasync be sure to copy or rename
the file to sasync.conf, which is the file that the sasync script looks for
- You can play with config settings quite a bit to customize how sasync runs

SASDIR="/opt/sasync"
  - Location of the sasync script
  - The default location is /opt/sasync. You can put sasync anywhere you like and/or run multiple, modified instances

SETDIR="$SASDIR/sasets"
  - Location of your set files
  - The default location is /opt/sasync/sasets. Again, relocate or rename your set files as you wish

JSDIR="/opt/sa"
  - Location of DIR with SA .json files. No trailing slash
  - This is the folder where you keep your service account json files. The should be numbered sequentially 1.json, 2.json etc

MINJS=1
  - First json file number in your JSON directory (JSDIR)
  - This is the number of the first json in your json set
  - For sasync to run correctly your jsons need to be sequential between MIN and MAX
  - You do not need to start at 1.json. For example you can use MINJS=100.json and MAXJS=200.json

MAXJS=1200
  - SET MAXJS TO THE MAX JSON FILE IN YOUR SET
  - After sasync uses this service account it will cycle back to MINJS.json

JSCOUNT="$SASDIR/json.count"
  - Location of json.count file (NOT the jsons themselves)
  - This file keeps track of which json has been last used. sasync uses a file rather than a variable to keep count
  in order to allow multiple/parallel instances of sasync to run while using the same range of jsons

NEXTJS=1
  - Cycle json count by NEXTJS. Default increment is 1
  - Using NEXTJS=101 pushes each cycle into a new project and may help avoid api issues 

FILTER="$SASDIR/filter"
  - Location of the default filter file
  - You can leave this file blank, but do not delete it as rclone looks for it
  - Filter files work a bit  differently than --include/--exclude. See https://rclone.org/filtering/#filter-from-read-filtering-patterns-from-a-file

NOTIF_ALL=false or true
  - Use apprise to send ALL messages
  - Must install/config apprise separately)

NOTIF_KEY=false or true
  - Use apprise to send only summary/key messages
  - Must install/config apprise separately)

NOTIF_COMM="apprise"   
  - "apprise" sends messages to the default service           
  - Adjust the apprise command if you want to send to a non-default service or to use options

IFS1=$' \t\n'
  - Field separator for set file
  - Use ',' or '|' if you do not use space


<br>

**Flags to control which checks and what cleaning is done. All = false works in standard case**

CHECK_REMOTES=true
  - Check if remotes are configured in rclone. Generally good to run this with new set files
  - sasync runs faster if this is set to false by avoiding rclone lsd, rclone size etc

CALC_SIZE=false
  - Checks the size of the source and destination folders in the current set pair, then calculates the difference
  - Using --maxtransfer from the set file, estimates the number of service accounts (SAs) required to copy/sync the difference
  - The estimated number of SAs acts as an upper limit to the number that sasync will use before moving to the next pair in the set file
  - If source and destination size are identical then tells sasync to skip to the next pair in the set file

DIFF_LIMIT=95  
  - Use a number from 0 to 100, no % in the variable
  - For DIFF_LIMIT to work it needs CALC_SIZE=true
  - If src/dest size is less than or equal to the DIFF_LIMIT % then skip the current pair and move to the next pair

FILE_COMPARE=false
  - Runs hash check against files using `rclone check` to identify if source and destination are identical
  - If source and destination files are identical then tells sasync to skip to the next pair in the set file

CLEAN_DEST=false
  - Set to true if you want to clean the destination
  - Runs rclone to dedupe, delete trash and remove empty directories
  - This can take quite a it of time with large folders

CLEAN_SRC=false
  - Set to true if you want to clean the source
  - Same function as CLEAN_DEST but for source folders

PRE_CLEAN_TDS=false
  - Set to true to clean remotes fore running rclone
  - If you believe you may have a lot of duplicate files or overlapping duplicate folders then it can be useful
   to clean up before syncing

EXIT_ON_BAD_PAIR=false
  - Exit (true) or continue (false) if set pair is bad
  - When sasync runs rc_check and finds a source or destination that is 'bad' this flag determines if sasync
  exits or moves on to the next set pair

SRC_LIMIT=
  - Daily outbound limit in GB. Blank if none. For this flag to work CALC_SIZE must be true
  - This can be useful when your source is not a TD, thus you cannot use multiple SAs to gain quota
  - For a single google account the daily download quota is 10TB/day (bear in mind any quota you may use for streaming or other activities)

BAK_FILES=false (true/false)
  - Send files to backup dir rather than delete them

BAK_DIR=backup
  - Backup files sent to destination `backup`. Change if you prefer a different directory
  - rclone can only move files to a backup directory in the same remote, not to a different remote

MAKE_DESTDIR=false (true/false)
  - If directories are missing in the destination remote, set MAKE_DESTDIR=true and it will create them
  
**These global flags are applied to all sets when rclone runs. Tweak them as you like.**
  - Some flags and values may change from version to version

FLAGS="
FLAGS="
  --fast-list
  --checkers=32
  --tpslimit=4
  --tpslimit-burst=20
  -vP
  --stats=10s
  --max-backlog=2000000
  --ignore-case
  --size-only
  --no-update-modtime
  --drive-chunk-size=128M
  --drive-use-trash=false
  --filter-from="$FILTER"
  --track-renames
  --use-mmap
  --drive-server-side-across-configs=true
  --drive-stop-on-upload-limit
  "
  "

Check https://rclone.org/flags for detailed descriptions of flags

"--drive-stop-on-upload-limit"
  - this flag stops a single rclone cycle when Google signals that account or SA quota is reached

"--drive-server-side-across-configs=true"
  - To enable server side copying this flag must be true (for current versions of rclone; may change in future)
  - set to false if you want to force rclone to download/upload files
  - you can also add `--disable copy` to the flags here to disable all server side copying

"--fast-list"

  - fast list can speed up scans and reduce api calls
  - When used with rclone copy fast list sometimes creates duplicate files in the destination. Can be a good idea to
    set CLEAN_DEST to true if this happens to you

"-vP"

  - The v flag provides moderate feedback from the rclone command. Remove v for less feedback or use vv or  vvv for debugging
  - P shows progress for each cycle of rclone. Remove if you don't need to see this output

"--stats=5s"
  - Used in conjunction with the P flag. Omit or make longer for higher/lower frequency of progress updates

"--max-backlog=2000000"
  - Default for showing how many files are being copied is 10000. This allows seeing more files in the queue

"--ignore-case"
  - Used in conjunction with --filter-from. If using no filters then you can ignore/delete this flag

"--no-update-modtime"
  - Tells rclone to not update files if only the modtime has changed. Can speed up syncs when modtime is not important
  - Omit if you want modtimes to be updated

"--drive-chunk-size=128M"
  - Ignored for server side copy/sync
  - Adjust as you like depending on --transfers and available RAM

"--drive-use-trash=false"
  - This flag set to false tells rclone to permanently delete files rather than moving them to Trash
  - Change to true if you want files to stay in Trash (30 day default for Google Drive)

"--filter-from=$FILTER"
  - Default tells rclone to use any +/- include/exclude filters in the filter file
  - You can safely delete this line and sasync will run without filters

"--use-mmap
  - Helps with memory management on some systems

<br>

###  Tips and Notes

- [ MANUAL DIAGNOSTIC FOR CHECKING PERMISSIONS ON RCLONE REMOTES WITH SAs ](https://gitlab.com/88lex/sa-guide/-/blob/master/rclone_remote_permissions.md#manual-diagnostic-for-checking-permissions-on-a-rclone-remote)


- sasync can be run with crontab or as a task in windows

- sasync runs even faster if you use --max-age=1d or 1w. This is especially handy when running with crontab

- sasync is designed to work with Google Drive as well as local drives as the source. It may work with other rclone remote types but has been tested mostly with gdrive.
- When upgrading you can overwrite local files/changes by running
  >`git fetch`\
  `git reset --hard origin/develop`\
  `cp sasync.conf.default sasync.conf`\
  `chmod +x sasync`\
  `./sasync set.p8`\
  - Then edit sasync.conf

- Check each line in the sasync.conf file, especially the directory (`DIR`) locations. sasync will not run correctly if it cannot find sets, jsons or filters

- The default set file folder is `/opt/sasync/sasets`. If you want to put set files in another folder be sure to change sasync.conf SETDIR=/new/location

- sasync will NOT create base folders. e.g. If you specify `remote:movies` in a set file, you must create the `movies` folder in `remote:` before syncing
  - Example set pair: `sync my_video:media/kids bak_video:media/kids` If /media exists but the `kids` folder does not then rclone does not create the `kids` folder

- You may consider creating your own private repo called `sasets` in github/lab/bucket that contains your personal set files
Set files are easy to edit in github/gitlab and can be easily added to / updated to any machines running sasync

- There are a number of small utilities in the `utils` folder. They do things like install sasync, update rclone, clean set files, remove all comments

- Versions of rclone from 1.48 require a new flag `--drive-server-side-across-configs` in order to do server-side sync/copy. This flag has been added to sasync as a default. If you are running an older version of rclone or wish to not execute server-side copies simply delete the flag from sasync.conf

- Resource usage for this script is very light as the copy/move actions are all executed server side. That said, the script can be modified to use the --disable move,copy flag if you prefer, in which case I/O and CPU usage would rise

<br>

## **Files in the repo**
- **`sasync`** is the main script that pulls sync sets from 'set' files and runs rclone sync/copy/move.
- **`sasync.conf.default`** contains major variables and rclone global flags.
- **`rccheck`** checks rclone remote status
- **`sacalc`** estimates the number of service accounts required to sync/copy the gap between source and destination.
- **`sainit`** initializes working variables as well as log and backup folders
- **`cleantds`** deletes exact dupes, removes empty directories and permanently deletes trash. Can be toggled for destination and or source.
- **`json.count`** is an external counter for the json set. This allows running multiple instances of sasync that pull in
sequence from the jsons without wasting or duplicating usage.
- Two sample set files with rclone source, destination and max-transfer size. **`set.tv.sample`** and **`set.movies.sample`**
- **`filter`** contain file patterns to include or exclude.
- **`msg`** contains messages called by the script
- **`utils` folder** contains a few misc scripts.
  - `readsets` will check if your set.* files are readable by sasync.
  - `cleansets` will replace tabs and unreadable characters in your set.* files. Syntax is `./cleansets set.video`.
  - `install_sasync`** in /utils folder will automate some installation tasks for you. It is intended for use during initial install, but can be used with subsequent installs.


[##click to jump](#xxx)

<details>
  <summary>Click to expand!</summary>

  ## xxx
  1. A numbered
  2. list
     - With some
     - Sub bullets
</details>

---
<br>

###  Changelog V3.4
- [NEW] Added notifications using `apprise`.
  - NOTE: Must install and configure `apprise` for this to work.
  - There are two levels of notification. KEY sends start, stop and error messages. ALL pushes more detail. 
  - Toogle in `sasync.conf`. NOTIF_KEY=true/false. NOTIF_ALL=true/false.
  - The default command is `apprise`, which sends to your default apprise destination.
  - Notification apps other than apprise may work but have not been tested.
- [FIXED] COUNT fixed to work properly with NEXTJS other than 1. Use 101 to jump between projects with each SA cycle.


###  Changelog V3.2
Major change: If your prior version is 3.0 or lower then to run this new version of sasync correctly
you MUST remove the max transfer field (e.g. 600G) from your old set files.\
Suggested method is to copy your set files from /opt/sasync/sasets to /opt/sasync/sets,
then run the ./remove_maxt script for set.* files in the /sets folder.\
For now it's a good idea to keep the old set files (until the new rclone flag is fully tested).

- [NEW] If SOURCE is empty will skip that pair and create an error in a file `*_fail.log`
- [NEW] Added a variable called `DIFF_LIMIT` in sasync.conf.default (be sure to copy to your sasync.conf file)
  - `DIFF_LIMIT` is an INTEGER representing the ratio % of SOURCE / DESTINATION remote size.
  - IMPORTANT: `DIFF_LIMIT` only works if `CALC_SIZE=true`. You cannot calculate a ratio without sizes.
  - Examples:
    - If source remote is 20GB and destination remote is 10GB then the ratio would be 200 (200%, but we leave out the %).
    - If source remote is 5GB and destination remote is 10GB then the ratio would be 50 (50%, omitting %).
    - If you set `DIFF_LIMIT=70` then the first example would pass, but the second would fail.
    - On failure of the DIFF_LIMIT test sasync will write an error to `*_fail.log`
- [NEW] Added a file called `*_fail.log`. This file will be printed at the end of each sasync run. You can tail or monitor these files, or you can periodically send them to your email, bots, discord, etc.
- [NEW] Added a variable called `NEXTJS` in sasync.conf.default (be sure to copy to your sasync.conf file)
  - `NEXTJS` is the amount by which JSCOUNT (your SAs) will be increased with each cycle.
  - In most cases leaving `NEXTJS` at a default of 1 is fine.
  - In cases where you may have errors from too many api calls and/or are running multiple parallel instances of sasync,
  it is possible that setting `NEXTJS=101` can help reduce errors.
  - Each GSuite project has its own api quota. Incrementing by 101 forces parallel sasync instances to use a different project's quota.
  - Note that setting `NEXTJS=101` only works well if you have multiple projects and many SAs.
- [CHANGED-TEMP] For `CLEAN_DEST` and `CLEAN_SRC` temporarily replaced `rclone delete remote: --drive-trashed-only` with `rclone cleanup remote:`.
  - The delete remote with --drive-trashed-only has been hanging sometimes with recent rclone releases. And in principle rclone cleanup should work, even if not immediately. Will monitor and change back in future if needed.

###  Changelog V3.1

- [NEW] Added --drive-stop-on-upload-limit to sasync.conf
- [REMOVED] --max-transfer logic and set.file requirement
- [CHANGED] Default location of set files from /sasets to /sets
- [NEW] Added a script (remove_maxt) in /sets folder to remove max_transfer variable from set files
  - Best way to do this is `cp -r /opt/sasync/sasets /opt/sasync/sets` then
  - `cd /opt/sasync/sets && chmod +x remove_maxt` then `./remove_maxt set.*`
- [REMOVED] IFS auto sense. Forces user to choose separator in sasync.conf file. e.g. IFS1=' ' or IFS1=','
- [REMOVED] TMOUT (timeout) flag and code
- [ADDED] Link for new instructions for manually checking rclone remote permissions that impact successful running of sasync. ==> [ MANUAL DIAGNOSTIC FOR CHECKING PERMISSIONS ON RCLONE REMOTES WITH SAs ](https://github.com/88lex/sa-guide/blob/master/rclone_remote_permissions.md)

###  Changelog  V3.0

- [NEW] Added `-p number` option to run 'number' of parallel instances of sasync. Usage: For 3 parallel instances `./sasync -p 3 set.file`
  - Requires tmux and parallel apps. Install with command: `sudo apt install parallel tmux`
  - Sends sasync to n tmux windows. App will provide a link to open tmux
  - Can run a single instance of sasync in tmux with `./sasync -p 1 set.file`
  - When one set pair finishes next one auto loads
  - sasync works as usual without tmux or parallel when running `./sasync set.file`

- [NEW] Added -t option to run sasync in tmux. `./sasync -t set.file`

- [NEW-AGAIN] Added ability to run with multiple set files
  - `./sasync -p n set.file1 set.file2 set.file3`
  - Will merge set files then run in n tmux windows
  - When one set pair finishes next one auto loads
  - Works without `-p` running in main terminal one set pair at a time

- [UNCHANGED] sasync still supports custom config files as well as infinite rclone flags at the end, along with new -p options.
  - `./sasync -c my.conf set.file1 --flag1--flag2 -v --dry-run`

- [NEW] Allows creating missing directories in the destination remote with MAKE_DESTDIR [Deault: false]
  - Be careful with this option. If your remote does not exist it could create directories on your local disk

- ~~[NEW] Changed remote check from `rclone lsd` to `rclone about`. Should make it much quicker~~

- [NEW] Added `--drive-service-account-file` to READ/WRITE/DELETE checks. Should make check more reliable

- [ADDED] Added `csl` to alias shortcuts to navigate to logs directory
  - To access aliases run `cd /opt/sasync/utils;chmod +x add_aliases;./add_aliases`. Works on next boot
  - sas='/opt/sasync/sasync'
  - csa='cd /opt/sasync'
  - cse='cd /opt/sasync/sasets'
  - csl='cd /opt/sasync/logs'
  - csu='cd /opt/sasync/utils'
  - c..='cd ..'
  - t0='tmux a -t 0'

###  Changelog  V2.8

- [NEW] Added RW Read/Write check for destination when you enable rccheck [default=true]

- [NEW] New script `add_aliases` which will add shortcuts to move around sasync folders easily.
  - `csa` = cd /opt/sasync
  - `cse` = cd /opt/sasync/sasets
  - `csu` = cd /opt/sasync/utils
  - `sas` = will run sasync. So `sas set.bak` rather than `./sasync set.bak`
  - `c..` = cd ..
  - `t0`  = tmux a -t 0

- [NEW] ~~Trying to accommodate set files with various formats. At the moment you should be able to separate your fields with spaces, tabs, commas[,] or vertical bars[|].~~
  - ~~Use only one type of field separator. Do not mix separators in the same file - results may be unpredictable.~~
  - ~~One method to change space separators to ',' in your set file is to run `sed -r 's/\s+/,/g' set.file > set.file.new` in a bash terminal.~~

- [NEW] sasync can now do sync/copy without SAs in source or destination.
  - This is not the primary use-case, but if you want to include syncs to destinations where you do not have SA
  access or where SAs do not work (like My Drive) you can include that pair in your set file.
  - NOTE: Without SAs the sync cannot exceed whatever quota your single source or destination account has.

- [NEW] Optional backup of old files to separate directory, rather than deletion [default=false]. Uses --backup-dir rclone flag.
  - If source is root of remote `:` then no backup is made. rclone does not allow backups from root level.

- [NEW] Handles missing jsons, skips to next existing json

- [IMPROVED] Better checking of remote configs, service accounts and read/write status of destination

- [UPDATED] sasync should work with encrypted remotes now, but only when both source and destination have service accounts/SAs. [You MUST turn rccheck and sacalc to `false` to allow encrypted sync to work with SAs. Also include `--disable copy` in the config flags, set file or the sasync command line.]

- [NEW] Added a couple of utilities in /utils folder. These are very similar to the rcgen.sh script that Max includes in his TD mount script.
  - `rc_add_remotes remotes.test` will create/update rclone TD remotes using a text file (remotes.test which includes a list
  of remote names and TD IDs.
  - `rc_add_remotes1 remotes.test existingremote` will pull client_id, client_secret and the token from an existing remote.
  - `rc_add_remote2 remotes.test` is interactive, and will let you choose if you want SAs, tokens or both.

v2.3 CHANGES:
- [NEW] rclone flag pass through in the command line ==> `sasync set1 --flag1 --flag2 --etc`
  - To enable flag pass through multi-sets in the command line were removed (`sasync set1 set2` no longer works).

- [NEW] Choose which config file to use with -c flag and filename (` -c file.conf `)

- [NEW] Option to use csv set files - sasync now handles commas and tabs in set files. This allows easy editing with excel/gsheets

- [NEW] Flag in config file to exit or continue when source-destination remotes are bad

- [NEW] `rccheck` checks if the source and destination are TDs, MDs, shared folders or local. This then determines if SAs are used for the source and adjusts flags accordingly (optional)

- [NEW] FILE_COMPARE compares individual files and file sizes, skips to next set pair if identical (optional)

- [NEW] SRC_LIMIT allows limiting a sasync copy to a fixed TB/GB for situations where you do not have SA access (thus using quota from a single account on source side)
  - Limit applies to EACH set pair. Be careful how you use it if you have multiple folders pulling from the same root source

- [CHANGED] The sasync/config_backup folder is changed to sasync/backup to allow for other backups in future

- [REMOVED] SWEEPER. No longer needed

- [REMOVED] SASYNC-DEST. sasync should now copy destination-only SA files correctly
