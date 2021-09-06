---
title: Staff Notifications
date: 2021-04-12 14:23
---

# Staff Notifications
[Source](https://wiki.inmotionhosting.com/index.php?title=Staff_Notifications)

System administration is required to notify staff during certain situations.

## Notifications Needed

* Outages
* Planned maintenance
* Password changes

## Outages

### Unexpected downtime for service

* Any downtime exceeding 7 minutes whould be broadcast to Support using #qos.
* Any downtime exceeding 15 minutes should have a
	[post-mortem](https://wiki.inmotionhosting.com/index.php?title=Post-Mortem_Reports)
* T3 is required to report these on the [internal blog](inmotionstatus.com)

### Unexpected downtime for a server

* Any downtime exceeding 10 minutes whould be broadcast to Support via Slack and
	have post-mortem filed
* T3 required to report these in [internal blog](inmotionstatus.com)
* **Downtime exceeding 2.5 hours must be reported via Slack or phone to the Systems Manager (Trey) at any time of day.**
	+ See [emergency](https://wiki.inmotionhosting.com/index.php?title=Emergency) for contact info
	+ Events that will take several hours can be relayed to Trey as soon as we
		know that it will take that long. 
	
### Network problems affecting one or both datacenters

* Any downtime should be broadcast to Support via Slack and post-mortem filed
	+ Continual updates through Slack or via email may be needed. 
	+ If unsure how to handle an emergency, [escalate as appropriate](https://wiki.inmotionhosting.com/index.php?title=Emergency)
	+ T3 is required to report these on the internal blog

## Planned Maintenance

For planned maintenance the following procedure should be followed:
1. **Before the change:** Email and/or broadcast via Slack with what is being done,
	 when, and what server(s) affected
2. **During the change:** Remind Support via Slack that the change is in process
3. **After the change:** Email and/or broadcast via Slack that change is complete

_Significant changes should have a blog post written by management or T3._

## Password Changes

If the root password changes:
* T3 should be notified at: serverreports@inmotionhosting.com
* Passwords should be distributed into relevant jumpstation accounts and be
	removed within 48 hours.
If WHM and System passwords change:
* T2C, T2S, and T3 should be notified via tier2-int@inmotionhosting.com
* Passwords should be distributed into relevant jumpstation accounts

**Note: there is almost no reason that T2S should be making password changes; if T3 is available they should handle this. Please escalate to T3 through usual means.**

