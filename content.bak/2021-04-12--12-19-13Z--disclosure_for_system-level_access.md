---
title: Disclosure for System-Level Access
date: 2021-04-12 12:19
---

## Disclosure for System-Level Access
[Source](https://wiki.inmotionhosting.com/index.php?title=Disclosure_for_System-Level_Access)

### Responsibility/Accountability

If you make a mistake notify management or T3 immediately. No exceptions.
Come clean, be honest.

### Software Installation/Usage

* Don't install unapproved software
* Don't run unapproved software as root, or with sudo powers
* All software installations must be reviewed/approved/deployed by T3.
* In general if you run anything as root it absolutely must be an approved tool.

### Loops and Complex Commands

* Always echo loops before running, like so:
```
for user in $( cat accounts.txt ); do
  echo /scripts/killacct "$user"
done

# => /scripts/killacct userna5
# => /scripts/killacct userna6
# etc...
```

* Make sure complex commands are safe to run. It's ok to provision a VPS to test
	a command, if necessary. 
* Make sure you're running commands, scripts, etc on the right server, in the
	right directory, and in the right screen session.

### Scripts From Untrusted Sources

* Don't download and run code from untrusted sources or that hasn't been
	approved.
* Simple code snippets <50 lines should be reviewed thoroughly before running.
	+ Probably still better to get approval first though...
* Submit handy tools, code, or scripts to T3 to be reviewed and deployed
	properly.

### Cron Jobs

* Don't schedule unapproved cron jobs to run as root on shared.
* Admins may add entries to their individual user crontab or to a customer's
	crontab if there's a valid motivation to do so.		
	+ crontab entries should not escalate privs with `sudo` or `su`.

### Unattended or Long Running Processes

* Don't leave processes running unattended for long durations.
	+ Exceptions are backups, clean scripts, `imh-scan` might be an example. 

### Command History

* Do not clear the `bash` `history` at `/root/.bash_history`. 
	+ The only exception is to hide a password passed at the CLI, for security
		purposes. 
	+ If you must delete something from the history:
		1. Run `history` to get a numbered list of the history entries.
		2. Run `history -d $line_number` to delete that line from `history`.
