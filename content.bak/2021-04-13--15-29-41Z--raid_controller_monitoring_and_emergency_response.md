---
title: RAID Controller Monitoring and Emergency Response
date: 2021-04-13 15:29
---

# RAID Controller Monitoring and Emergency Response
[Source](https://wiki.inmotionhosting.com/index.php?title=Raid_Controller_Monitoring_and_Emergency_Response)
## Installing and Configuring RAID Monitoring

* [Install net-snmp](https://wiki.inmotionhosting.com/index.php?title=Install_net-snmp)
* [Nagios SNMP RAID Monitoring Setup](https://wiki.inmotionhosting.com/index.php?title=Nagios_SNMP_Raid_Monitoring_Setup)
* [Nagios-addcheckplugin](https://wiki.inmotionhosting.com/index.php?title=Nagios-addcheckplugin)

## Determining RAID Controller

Use this command to determine which RAID controller you're working with:
```
/sbin/lspci -vvv | grep -i RAID
```

Occasionally you may need to expand the search for SAS or PERC:
```
/sbin/lspci -vvv | grep -iE -B1 "RAID|PERC|SAS"
```

Once you've determined what RAID controller is being used, follow the relevant
guides found [here](https://wiki.inmotionhosting.com/index.php?title=Raid_Controller_Monitoring_and_Emergency_Response)
_(guide far too long to notate here)_
