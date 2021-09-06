---
title: Service Restarts
date: 2021-04-12 15:04
---

# Service Restarts
[Source](https://wiki.inmotionhosting.com/index.php?title=Service_Restarts)

In many cases, it's necessary to restart or reload the service. Mainly, if the
service becomes unresponsive or a configuration change is made.

T2S is allowed to restart almost any service on Shared/VPS/Node/Dedi, at their
discretion.

* If you expect the service to be down for an extended period of time, [notify staff](https://wiki.inmotionhosting.com/index.php?title=Staff_Notifications_-_Systems)
* If the service is listed below, special considerations need to be made:
	+ Named/BIND on Nameservers (ns1, ns2, etc)
		- This can take as long as 30 mins to restart and causes "cluster
			congestion." If possible do a rndc reload instead of restarting. 
		- check with T3 if you feel a restart of this service might be needed.
		- **Rarely does this service need to be restarted.**
	+ MySQL on Shared Servers
		- It can take over 5 mins to restart MySQL. MySQL restarts should be avoided
			if possible.
	+ Virtuozzo
		- Requires T3 approval since this starts/stops all containers on node. 
		- Can cause significant downtime and high load on machine. 

## Server Reboots

Most of the Shared or VP Node reboots need to be approved by T3. Linux very
rarely needs reboots. A reboot should be the last resort. If a server is
unresponsive, frozen, kernel panic'd, then T3 won't be able to do much to fix,
so a reboot is allowed. If you're not sure, escalate to leadership. 
* Every reboot must have a post-mortem. 
* You should be able to confirm through DRAC if the server is kernel panic'ing.
