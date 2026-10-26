# pdbackup

[![Project Status: Active](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active) [![License](https://img.shields.io/github/license/torfinn-brokke/pdbackup)](https://github.com/torfinn-brokke/pdbackup/blob/main/LICENSE) [![Last Commit](https://img.shields.io/github/last-commit/torfinn-brokke/pdbackup)](https://github.com/torfinn-brokke/pdbackup/commits/main) [![ShellCheck](https://github.com/torfinn-brokke/pdbackup/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/torfinn-brokke/pdbackup/actions/workflows/shellcheck.yml)

This is a Bash script for backing up a local folder to a remote Proton Drive folder.

It takes exactly two arguments: the local (source) folder and the remote (destination) folder.

## Requirements

`pdbackup` requires the `proton-drive` CLI to be installed on your system.

To check whether it is already installed, run the following command:

```bash
$ command -v proton-drive
```

If it is already installed, this command will indicate where it is installed. If the command returns nothing, you do not have `proton-drive` installed. The script will also check for `proton-drive` when called, and will exit with an error message if it cannot be found.

For installation instructions, consult [the Proton Drive CLI web page](https://proton.me/support/drive-cli).

## Installation

Clone the repository:

```bash
$ git clone https://github.com/torfinn-brokke/pdbackup.git
```

Copy the file `pdbackup` to your `~/bin` folder:

```bash
$ cp pdbackup ~/bin/
```

> **NOTE:** the above assumes that `~/bin` is in your `$PATH` environment variable. Check it with the command `echo $PATH`. If it is not, you can either add it (using the command `export PATH="$HOME/bin:$PATH"`) or make a symlink into a directory that is already on the path.

Make the file executable:

```bash
$ chmod +x ~/bin/pdbackup
```

## Usage

`pdbackup` is meant to simplify the use of the proton-drive CLI for keeping a local folder backed up to a remote folder on Proton Drive, either manually or as part of a systemd timer or user `crontab`.

The script takes two arguments: the local folder and the remote folder. Here is an example:

```bash
$ pdbackup "~/My Local Documents" mybackup
```

> **NOTE:** Paths with spaces in them must be quoted. Do not add the Proton Drive root folder (`/my-files`) to the remote path; this is done automatically by the script.

The log file `pdbackup.log` is created in a local directory chosen by the user, by default the user's home folder (this can be changed by editing the constant *LOG_TARGET*).

When called, the script tests the network connection and notifies the user if there is an error. Then it tests whether the local path exists, and exits with an error if it does not.

The script then starts uploading the files to a remote folder. The date, in the format YYYY-MM-DD, is appended to the remote folder name to make it easier to locate a given backup. In the example above, the remote folder name `mybackup` might become `mybackup_2026-10-26`.

If unable to upload, the script exits with an error message and writes to `pdbackup.log`. If successful, a success message will be sent to `pdbackup.log`.

For help, use:

```bash
$ pdbackup -h
```

In this repo there are also two other  files, `pdbackup.service` and `pdbackup.timer`, that can be used for setting up `pdbackup` as a system service.

> **NOTE:** This is for advanced users only. If you are not comfortable with setting up system timers, use `crontab` or similar instead.

To do this, first edit `pdbackup.service` and find the field `User=`. Change `<username>` to the user name of your choice. The default interval of twice a month can also be changed by editing the field `OnCalendar=` in `pdbackup.timer`.

Then copy both files to `/etc/systemd/system/` and run the following two commands:

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now pdbackup.timer
```

Then check that you get the statuses "loaded", "enabled" and "waiting" when you issue the command  `systemctl status pdbackup.timer`, and that you can see the next run time with `systemctl list-timers pdbackup.timer`.

## Notes

* Always perform a manual check to ensure the script works as intended before relying upon it for automated backups.
* When using `pdbackup` for automated backups, be sure to check `pdbackup.log` regularly to make sure the script performs as intended.

## License

[MIT](https://choosealicense.com/licenses/mit/)
