---
title: Post-Mortem Reports
date: 2021-04-12 15:19
---

# Post-Mortem Reports
[Source](https://wiki.inmotionhosting.com/index.php?title=Post-Mortem_Reports)

## General Procedure

A post-mortem is required whenever a server or specific service is impacted for
an extended period of time as defined in the [staff notification](https://wiki.inmotionhosting.com/index.php?title=Staff_Notifications) policy.
* The person on Nagios files the post-mortem
* Management reviews the post-mortem and after reviewing the incident as well as
	trends, will take any follow-up action needed
* It's important that the information be as complete as possible and the person
	handling Icinga to show relevant logs as well as anything not logged that
	might be useful information, and only available at the time of the incident.


The post-mortem should ideally contain the following information:
* What server(s) had the outage
* What service(s) were impacted
* When the problem occured and when/if it was resolved
* Any escalation paths that were taken, who was involved in troubleshooting,
	whether there was a hand-off, etc.
* What the overall problem was (Memory, CPU, network-related, etc)
* What underlying issue caused the overall problem
* What steps are being taken to prevent the issue from re-occurring

## Timely and Immediate items to be logged

The most interesting/useful data can be found around the time of the event.
Information that might be relevant:
* check_user
* recent-cp
* check_vps 
* vzlist -Ho veid,laverage | sort -nk2 | tail 
* Any info from other tools such as `iotop` and `vzstat`.

## Common Resource Bottlenecks

The post-mortem report should contain:
* Your diagnosis
* What caused it
* and why 

### sar
Allows you to go back in tima and see exactly what happened at a specific
moment. 

`sar`

Common flags:
* `-q` Report queue length and load averages
* `-r` memory and swap
*	`-p` cpu
* `-b` disk/block transactions
* `-n DEV` network traffic by DEVice 

You can use `-s` and `-e` to define start/end times you want data from:
`sar -q -s "14:25:00" -e "15:30:00"`

Using `pastebin` you can paste useful data:
* Load averages with CPU usage:
	`( sar; echo -e "\n"; sar -q ) | pastebin`
* 1m load average with memory usage:
	`( sar -q; awk '{print $5}'; echo -e "\n"; sar -r ) | pastebin` 
* memory usage with I/O wait%:
	`( sar -r; echo -e "\n"; sar | awk '{print $7}' ) | pastebin`
* memory usage with I/O wait% and load averages:
	`( sar -r;  echo -e "\n"; sar | awk '{ print $7 }';  echo -e "\n"; sar -q | awk '{ print $5,$6,$7 }' ) | pastebin`
* disk transactions, 1m load average, and iowait%:
	`( sar -b;  echo -e "\n";  sar -q | awk '{ print $5 }';  echo -e "\n";  sar | awk '{ print $7 }' ) | pastebin`
* disk transactions, 1m load average, iowait% and swapused%:
	`( sar -b;  echo -e "\n";  sar -q | awk '{ print $5 }';  echo -e "\n";  sar |	awk '{ print $7 }';  echo -e "\n";  sar -r | awk '{ print $10 }' ) | pastebin`

### toplogs/dcpumon

cPanel takes a snapshot of the top processes every 5 minutes and saves it in a
"toplog." 
Toplogs are located in: `/var/log/dcpumon/`

### graphite

* Data populated every minute
* CPU, I/O, # of procs, network stats, etc
[graphite](http://graphite.imhadmin.net/)
[docs](https://wiki.inmotionhosting.com/index.php?title=Graphite#Creating_and_Viewing_Metrics)

### swap dumps

Due to servers swapping and crashing, nagios has been configured to collect
`top` and `ps` immediately after recording a delta of >4% in swap consumption.
Check these for additional data:
```
top -b -n1 -c > /home/nagios/check_swap.topdump
ps auwx | sort -nk4 > /home/nagios/check_swap.psdump
```
_Note: VP nodes also include a `ovzstatus.pl` dump:_
```
/opt/vpsrads/ovzstatus.pl | sort -nk4 > /home/nagios/check_swap.ovzdump
```

## Identifying problematic users

### process accounting

If a user process has crashed a server, process accounting should have recoreded
it and saved data. 
* First run `sa -m` to see if anyone stands out as high usage after server is
	back on-line.
	```
	sa -m | sort -nk4 | tail -10
	```
* If you see a user with a cp count higher than 100, they may have been
	responsible for crashing the server. Use `lastcomm` to extract the process
	data recorded for that user.
	```
	lastcomm $user
	```

### RADS tools
If you feel the incident was user-caused, there's a variety of
[RADS](https://wiki.inmotionhosting.com/index.php?title=RADS) tools to include in your report.

## Evaluating Server Hardware

### DRAC
_Dell's Remote Access Card_
Bad hardware will be logged in the DRAC system log.

### RAID
If a drive is failing in a RAID array, it can impact server performance. Nagios
is alerted for degraded RAIDs. Errors from a drive will be reported to the raid
controller (raid card) and visible in the [RAID
utility](https://wiki.inmotionhosting.com/index.php?title=Raid_Controller_Monitoring_and_Emergency_Response).

### dmidecode
`dmidecode` is used to talk directly with the harware. 
Run `dmidecode | less` and search for "error" and "DIMM".

## Review Kernel Logs

Kernel logs can sometimes be the only indication of what might have gone wrong.
If you have yet to determine a clear cause for something, you can begin
reviewing the kernel logs at `/var/log/kernel.log` or `dmesg`.

You can ignore things like this, they're irrelevant:
```
** RABHIT ** IN=eth0 OUT= MAC=84:2b:2b:4c:0a:90:00:01:e8:1a:0c:2e:08:00 SRC=64.234.252.122 DST=70.39.232.97 LEN=40 TOS=0x00 PREC=0x00 TTL=235 ID=10016 PROTO=TCP SPT=113 DPT=35218 WINDOW=0 RES=0x00 ACK RST FIN URGP=0 
TCP: Treason uncloaked! Peer 85.74.145.179:58413/80 shrinks window 1291433611:1291434868. Repaired.
```

More serious messages such as hardware, I/O errors, or segmentation faults may
be of interest. For example:
```
[ 45.750008] sd 0:0:0:0: [sda] Sense Key : Hardware Error [current]=20
[ 45.750012] sd 0:0:0:0: [sda] Add. Sense: Internal target failure
[ 45.750017] end_request: I/O error, dev sda, sector 721945599
[ 45.750079] Buffer I/O error on device dm-0, logical block 90243144
[ 45.750137] lost page write due to I/O error on dm-0
[ 87.970296] sd 0:0:0:0: [sda] Add. Sense: Internal target failure
[ 87.970302] end_request: I/O error, dev sda, sector 83324999
```
_Indicates hard drive and/or RAID error and should be immediately escalated to DC staff._

Segmentation faults can indicate bad code or a memory leak. This is normal from
time-to-time, but if there's an abundance of these, this could be an issue:
```
php[3014]: segfault at 00007fff655dbaa8 rip 0000003248042854 rsp 00007fff655dba70 error 6
php[24113]: segfault at 00007fffcc23cff0 rip 00000032480304eb rsp 00007fffcc23cfc8 error 6
php[18286]: segfault at 00007ffff29afad8 rip 0000003248042854 rsp 00007ffff29afaa0 error 6
php[28594]: segfault at 00007fff87b87c58 rip 000000324e046294 rsp 00007fff87b87c20 error 6
php[22148]: segfault at 00007fff1a81aff8 rip 000000324e049360 rsp 00007fff1a81b018 error 6
php[13285]: segfault at 00007fffa8adebb8 rip 0000003248042854 rsp 00007fffa8adeb80 error 6
```

## Submitting the Report

A quick and easy post-mortem report submission form can be found at the 
[eDesk Admin Portal](http://edesk.imhadmin.net/new/) or on the left-hand side of
**uDesk**. Use as many flags as appropriate to categorize the issue for tracking
trends.


## Behind the Scenes

Post-mortem system lives on cpjump at path: `/var/www/html/postmortem`
Flags are stored in the sqllite db and can be edited using sqlite3 or SQLAlchemy
For example:

```
# cd /var/www/html/postmortem
# /opt/imh-python/bin/python
```
```
>>> from core import db
>>> from models import Flag
>>> db.session.add(Flag(name='hardware'))
>>> db.session.add(Flag(name='ddos'))
>>> db.session.add(Flag(name='load'))
>>> db.session.add(Flag(name='panic'))
>>> db.session.commit()
```






























