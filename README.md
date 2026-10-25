# finlog

[![Project Status: Active](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active) [![License](https://img.shields.io/github/license/torfinn-brokke/finlog)](https://github.com/torfinn-brokke/finlog/blob/main/LICENSE) [![Last Commit](https://img.shields.io/github/last-commit/torfinn-brokke/finlog)](https://github.com/torfinn-brokke/finlog/commits/main) [![ShellCheck](https://github.com/torfinn-brokke/finlog/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/torfinn-brokke/finlog/actions/workflows/shellcheck.yml)

This is a Bash script for recording and listing personal expenses in a local SQLite database.

It can do the following:

* Add transactions to a local database.
* Delete transactions by transaction number.
* List transactions with optional filters (date, amount, category, and/or notes).
* Provide a summary of all stored transactions by category.

It is currently set up for Norwegian standards (DD.MM.YYYY date format, NOK as the currency and the comma (`,`) as decimal separator). finlog is designed for use with Linux, and uses `date -d`, which is GNU-specific, as part of its main logic.

> **NOTE:** this has been a learning project for me as part of getting to grips with Bash scripting, SQLite3, and using GitHub. Treat it like what it is: an experimental learning tool.

## Requirements

finlog requires SQLite3 to be installed on your system.

To check whether it is already installed, run the following command:

```bash
$ command -v sqlite3
```

If it is already installed, this command will indicate where it is installed, commonly `/usr/bin/sqlite3`. If the command returns nothing, you do not have SQLite3 installed.

To install it on Linux, use one of the following three commands depending on which Linux distribution you are on:

```bash
$ sudo apt install sqlite3 # Debian-based distributions (Ubuntu, Linux Mint...)
$ sudo dnf install sqlite  # Red Hat-based distributions (Fedora, RHEL...)
$ sudo pacman -S sqlite    # Arch Linux
```

If you are on another Linux distribution, or using Windows/macOS, consult the relevant documentation for how to install SQLite3.

## Installation

Clone the repository:

```bash
$ git clone https://github.com/torfinn-brokke/finlog.git
```

Copy the file `finlog` to your `~/bin` folder:

```bash
$ cp finlog ~/bin/
```

> **NOTE:** the above assumes that `~/bin` is in your `$PATH` environment variable. Check it with the command `echo $PATH`. If it is not, you can either add it (using the command `export PATH="$HOME/bin:$PATH"`) or make a symlink into a directory that is already on the path.

Make the file executable:

```bash
$ chmod +x ~/bin/finlog
```

## Usage

finlog is meant for personal finance tracking through adding expenses to (or delete them from) a database, from where they can be retrieved and listed by date, amount, category and any attached notes. The total expenses for the listed items is also displayed.

The first time it is used, finlog will create a SQLite3 database file in the directory `~/.local/share/finlog` (the directory will be created if it does not exist). Subsequent usage will access this same file as long as it exists and is readable. If the database cannot be created, or is unreadable, the script will exit with an error message.

> **NOTE:** finlog uses NOK (Norwegian krone) as its currency, and will prefix all amounts with "kr".

To add a transaction, use `finlog add [date] amount category [-n note]` in the following format:

* The date needs to be in the DD.MM.YYYY format. If no date is entered, today's date will be used automatically.
* The amount must contain no spaces and must use a comma (`,`) as the decimal separator.
* The category must be one word, without any whitespace.
* The optional note can be of any length and contain whitespace.

All input is validated. Invalid input will cause the script to exit with a description of what is wrong, as well as a description of how to get it right.

Here is an example of how to add a transaction, including the optional note:

```bash
$ finlog add 23.09.2026 1343,43 Groceries -n Bread and milk

[2026-09-24 18:06:56] Transaction successfully added:

┌─────┬────────────┬──────────────┬──────────────────┬───────────────────────────────┐
│ ID  │    Date    │    Amount    │     Category     │             Note              │
├─────┼────────────┼──────────────┼──────────────────┼───────────────────────────────┤
│ 5   │ 2026-09-23 │   kr 1343,43 │ Groceries        │ Bread and milk                │
└─────┴────────────┴──────────────┴──────────────────┴───────────────────────────────┘

$
```

To delete a transaction, use `finlog delete transaction_number` like this:

```bash
$ finlog delete 6

The following transaction will be deleted:

┌─────┬────────────┬──────────────┬──────────────────┬───────────────────────────────┐
│ ID  │    Date    │    Amount    │     Category     │             Note              │
├─────┼────────────┼──────────────┼──────────────────┼───────────────────────────────┤
│ 6   │ 2026-09-24 │    kr 454,43 │ Groceries        │ Snacks                        │
└─────┴────────────┴──────────────┴──────────────────┴───────────────────────────────┘

Are you sure? (y/n) > y

[2026-09-24 18:08:00] Transaction 6 successfully deleted.

$
```

To list all transactions in the database, or filter the results corresponding to a particular date, amount, category or note, use `finlog list` in the following manner:

```bash
$ finlog list all         # Lists all transactions added to the database.
$ finlog list -d [date]   # Only the DD.MM.YYYY format is accepted (no date input = today).
$ finlog list -a amount   # No spaces, comma as decimal separator, maximum of two decimals.
$ finlog list -c category # Wildcards: '%' matches multiple chars, '_' matches one char.
$ finlog list -n note     # Wildcards: '%' matches multiple chars, '_' matches one char.
```

Listing transactions will also make finlog display the total amount for all items listed. This can be used to display, for example, the total amount spent on a particular date. While it can also be used to display the total amount spent for a particular category, see the `finlog summary` section below.

Here is an example lookup:

```bash
$ finlog list -c Groc%

┌─────┬────────────┬──────────────┬──────────────────┬───────────────────────────────┐
│ ID  │    Date    │    Amount    │     Category     │             Note              │
├─────┼────────────┼──────────────┼──────────────────┼───────────────────────────────┤
│ 5   │ 2026-09-23 │   kr 1343,43 │ Groceries        │ Bread and milk                │
│ 7   │ 2026-08-20 │    kr 653,03 │ Groceries        │                               │
└─────┴────────────┴──────────────┴──────────────────┴───────────────────────────────┘

Sum Total: kr 1996,46

$
```

The last function of finlog, `finlog summary`, is to display the total amount spent per category, as well as the sum total of all added transactions.

Here is an example of how to use it:

```bash
$ finlog summary

┌──────────────────┬──────────────┐
│     Category     │    Total     │
├──────────────────┼──────────────┤
│ Entertainment    │    kr 745,00 │
│ Groceries        │   kr 1996,46 │
│ Utilities        │   kr 1430,00 │
└──────────────────┴──────────────┘

Sum Total: kr 4171,46

$
```

> **NOTE:** Be careful about spelling the category names the exact same way every time you add a transaction. Otherwise you might end up with, for example, one `Groceries` and one `grocery` category.

To back up the database you can copy the file `~/.local/share/finlog/finlog.db` to a secure location.

To uninstall, remove the script file from where you installed it, and delete the folder `~/.local/share/finlog`.

For help, use:

```bash
$ finlog -h
```

## Notes

* This project has mainly served as an exercise during my journey towards mastering Bash scripting.
* finlog is made for Norwegian standards when it comes to date format, decimal separators and amount prefixes.

## License

[MIT](https://choosealicense.com/licenses/mit/)
