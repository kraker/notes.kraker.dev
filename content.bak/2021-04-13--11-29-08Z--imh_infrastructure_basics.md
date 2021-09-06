---
title: IMH Infrastructure Basics
date: 2021-04-13 11:29
---

# IMH Infrastructure Basics
[Source](https://wiki.inmotionhosting.com/index.php?title=IMH_Infrastructure_Basics)

## The Datacenters

* Los Angeles, CA - powered by Corporate Colocation
* Ashburn, VA - powered by Equinix

For compliance info: https://wiki.inmotionhosting.com/index.php?title=SAS_70

## The Networks

Currently we have 3 bw lines in use.

### Public

**Bandwidth A:** Main network on WC with majority of shared, v-dedicated, dedi's
(CorporateColo and Level 3 using BGP)
**Bandwidth B:** No longer in use
**Bandwidth C:** EC network - [GTT](http://www.gtt.net/) and
[Level3](http://www.level3.com/) using BGP

### Private

Our Shared, VP, and backup server use private networks. EC and WC are separate
and can't communicate with each other.
* 192.168.x.x - WC
* 192.168.200.x - EC

#### West Coast Network

All WC servers are 10.128.0.0/16 on internal network
* 10.128.0.x - Internal Use
* 10.128.1.x - Internal Use
* 10.128.2.x - Backup Servers
* 10.128.10.x - VP Nodes
* 10.128.20.x - Hub Servers
* 10.128.100.x - Shared Servers
* 192.168.150.0 - Internal Use

#### East Coast Network

All EC servers are 10.129.0.0/16 on internal network
* 10.129.0 - Internal Use
* 10.129.1 - Internal Use
* 10.129.2 - Backup Server
* 10.129.10 - VP Nodes
* 10.129.20 - Shared Servers
* 10.129.100 - Shared Servers

## DNS

We have public, private and clustered DNS servers
* **Public NS:** 
	+ ns1.inmotionhosting.com, ns2.inmotionhosting.com
	+ ns1.webhostinghub.com, ns2.webhostinghub.com
* **Private Caching NS:** Preceeded with 'cache', such as cache1, cache2, etc.
	These are only used internally by servers for DNS lookups.
* **DNS Authority Servers:** Servers that validate requests and DNS changes to
	verify that only the owner can make a zone change. 
	+ ash-sys-pro-dns1.inmotionhosting.com
	+ ash-sys-pro-dns2.webhostinghub.com
* **Internal Zone Servers:** Allow us to update our own internal zones.
	+ ash-sys-pro-imhzone.inmotionhosting.com
	+ ash-sys-pro-hubzone.webhostinghub.com

_The [DNS authority](https://wiki.inmotionhosting.com/index.php?title=DNS_Authority) wiki has more in-depth explanation: 

## Internal Servers

A list of internal servers can be found at **IMH System Center >> Internal**
VPS ranges of 1000s and 10000s are reserved for internal use.

* **sys** - hosts secured internal apps (like PP). Only top managers/T3-II+ have
	access
* **mtr** - Monitoring servers, host applications use Nagios
* **vpimh** - VP Nodes that host core as well as internal VPS' used for various
	purposes (support center, forums, chat, etc).
* **ba** - Backup servers that house ahsred and VPS backups
* **voip** - Phone server
* **hotswap** - Non-permanenet servers, used for testing, available in case of
	emergency
* **MD1000/MD1200** - Not actually servers, but large backup arrays attached to
	backup servers to provide additional capacity. 
* **cache** - Caching nameservers, used internally for DNS lookups.
* Newer naming conventions can be found
	[here](https://wiki.inmotionhosting.com/index.php?title=Asset_Nomenclature_and_Naming_Conventions)
## The Software

Our servers use the following software
* CentOS
* Virtuozzo
* cPanel

## The Hardware

All shared servers are **Dell**
Some newer dedi's are **Lenovo**

_Details can be found in SC._

### Shared

**CPU**

Get number of processors:
```
cat /proc/cpuinfo | grep processor | wc -l
```

**MEM** 

```
cat /proc/meminfo
```
```
free -m
# or
free -g
# or 
free -h
```

**Disk Drives**
Drive numbers vary. All shared servers run [RAID 5](https://wiki.inmotionhosting.com/index.php?title=RAID#RAID_5) unless they're SSD then they run RAID 6. 

### VP Nodes

**CPU**
```
cat /proc/cpuinfo
```

**MEM**
```
cat /proc/meminfo
```
```
free -m
# or
free -g
# or 
free -h
```

Or you can run this from the node:
```
ovzstatus.pl | grep CTID
```

**Disk Drives**
Usually run RAID 10 if spinning media and RAID 6 if SSD.

### Dedicated Servers

Current plan specs: http://www.inmotionhosting.com/dedicated_servers.html

**CPU**
Or you can view specs in SC.

**MEM**
(Same as above)

**Hard Drives**
Only Elite and CC have RAID configured. It's RAID 1 for Elite CC500 and CC1000
servers. It's RAID 6 for CC2000 server.

Legacy specs can be found here: http://www.inmotionhosting.com/cheap-dedicated-servers

**Dedi Upgrades**
See [Hardware Upgrades](https://wiki.inmotionhosting.com/index.php?title=Hardware_Upgrades)
