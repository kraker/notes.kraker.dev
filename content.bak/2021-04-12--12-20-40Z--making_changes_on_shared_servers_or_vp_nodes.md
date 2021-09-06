---
title: Making Changes on Shared Servers or VP Nodes
date: 2021-04-12 12:20
---

# Making Changes on Shared Servers or VP Nodes
[Source](https://wiki.inmotionhosting.com/index.php?title=Making_Changes_on_Shared_Servers_or_VP_Nodes)
Don't make changes accross shared servers or VP nodes without explicit
permission from T3.

## Policy for Changes

Don't do anything below without explicit permission from T3:
* Server-wide software install/update/config
* Enable/Disable server-wide features
* Changes that will affect a large number of customers
* Unprecedented bug fixes for cPanel/WHM

Exceptions to the above are outlined in the wiki

## Requesting Server Changes

Process for handling change requests on shared servers:
1. Submit [Jira Ticket to T30](https://jira.imhdev.com/projects/T3O/issues)
2. T3 reviews request and investigates pros/cons; possible side-effects.
3. T3 approves
4. Plan is developed for rolling out change
5. Appropriate staff is
	 [notified](https://wiki.inmotionhosting.com/index.php?title=Staff_Notifications)
	 before/during/after the change.

## Deploying Server Changes

Any changes made to a production machine should be tested in a similar env. 

After testing, changes are generally pushed out in the following stages:
* 1 Server
* 5 Servers
* All Servers
