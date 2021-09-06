```
Config 2.1Known issues:
-no shutdown may still be required to remove shutdown command and enable interfaces 1 and 2 after reboot.
-See https://wiki.inmotionhosting.com/index.php?title=DC_Tech#Cisco_ASA_5506x_SETUP_UPDATED
-Need to add instructions to copy running config to startup config to make config retain through reboot.
-Credentials still need to be ran as separate commands, just as instructions in comments suggest.
-Removing or adding inspection for specific protocols can still be disabled/enabled by commenting out or removing # for protocols that cx requires.ASA Version 9.8(2)24
!
firewall transparent
#change the hostname to the server in question
hostname UPDATE_ded4697ASA
#IMH default PDU password used for both local and remote access for IMH employees only
#After config pasted in add Admin pass- enable password "examplepassword"
#After config pasted in add Systems user- username imhuser password "examplepassword" privilege 15
#Systems may need to create an additional user for cx to remotely adminnames
!
#Plug switch into this port
interface GigabitEthernet1/1
 no shutdown
 nameif uplink
 bridge-group 1
 security-level 0
!
#Plug cx into this port
interface GigabitEthernet1/2
 no shutdown
 nameif cx
 bridge-group 1
 security-level 100
!
interface GigabitEthernet1/3
 shutdown
 no nameif
 no security-level
!
interface GigabitEthernet1/4
 shutdown
 no nameif
 no security-level
! 
interface GigabitEthernet1/5
 shutdown
 no nameif
 no security-level
!  
interface GigabitEthernet1/6
 shutdown
 no nameif
 no security-level
!
interface GigabitEthernet1/7
 shutdown
 no nameif
 no security-level
! 
interface GigabitEthernet1/8
 shutdown
 no nameif
 no security-level
!  
#Management port remains shutdown, we do management inline
interface Management1/1
 management-only
 shutdown
 no nameif
 no security-level
!
#Bridge interface for use with the transparent ports.  IP address will be the management address.
interface BVI1
 ip address [ASA_IP] [ASA_IP_Subnet]
!
ftp mode passive
#Basic access-list.  These are whitelists, thus you must add rules for any service you add.
#Added exception for 5666 to allow icinga and avoid false positive alerts
access-list OUTSIDE extended permit tcp host 173.247.250.216 any eq 5666
access-list OUTSIDE extended permit icmp any any
#access-list cx extended permit icmp any any
access-list OUTSIDE extended permit tcp any any eq ftp
access-list cx extended permit tcp any any eq ftp-data
access-list OUTSIDE extended permit tcp any any eq ssh
access-list OUTSIDE extended permit tcp any any eq smtp
access-list OUTSIDE extended permit tcp any any eq domain
access-list OUTSIDE extended permit udp any any eq domain
access-list OUTSIDE extended permit tcp any any eq www
access-list OUTSIDE extended permit tcp any any eq pop3
access-list OUTSIDE extended permit tcp any any eq https
access-list OUTSIDE extended permit tcp any any eq 993
access-list OUTSIDE extended permit tcp any any eq 995
#access-list OUTSIDE extended permit udp any any eq 995
access-list OUTSIDE extended permit tcp any any eq 2083
access-list OUTSIDE extended permit tcp any any eq 2087
access-list OUTSIDE extended permit tcp any any eq 2096
access-list OUTSIDE extended permit udp any any eq 2096
access-list OUTSIDE extended permit tcp any any eq 465
access-list OUTSIDE extended permit tcp any any eq 587pager lines 24
mtu uplink 1500
mtu cx 1500
icmp unreachable rate-limit 1 burst-size 1
no asdm history enable
arp timeout 14400
no arp permit-nonconnected
#Apply the access-list to the transparent ports.  Note that without this, you'll have no connectivity.
access-group OUTSIDE in interface uplink
#Create a route so we can access our management port.
route uplink 0.0.0.0 0.0.0.0 [Gateway_IP_for_ASA_and_cxServer] 1
timeout xlate 3:00:00
timeout pat-xlate 0:00:30
timeout conn 1:00:00 half-closed 0:10:00 udp 0:02:00 icmp 0:00:02
timeout sunrpc 0:10:00 h323 0:05:00 h225 1:00:00 mgcp 0:05:00 mgcp-pat 0:05:00
timeout sip 0:30:00 sip_media 0:02:00 sip-invite 0:03:00 sip-disconnect 0:02:00
timeout sip-provisional-media 0:02:00 uauth 0:05:00 absolute
timeout tcp-proxy-reassembly 0:01:00
timeout floating-conn 0:00:00
#aaa and user-identity stuff to allow people to log into the switch.
user-identity default-domain LOCAL
aaa authentication ssh console LOCAL
aaa authorization exec LOCAL auto-enable
no snmp-server location
no snmp-server contact
service sw-reset-button
crypto ipsec security-association pmtu-aging infinite
crypto ca trustpool policy
telnet timeout 5
no ssh stricthostkeycheck
#Allow ssh access from our jumpstation
ssh 144.208.77.66 255.255.255.255 uplink
#Allow ssh access from server for cx (INSIDE) and add a new line for every server behind the firewall
ssh [cx_server_IP] 255.255.255.255 cx
#ssh [cx_server2_IP] 255.255.255.255 cx
#ssh [cx_server3_IP] 255.255.255.255 cx
#ssh [cx_server4_IP] 255.255.255.255 cx
ssh timeout 5
ssh key-exchange group dh-group1-sha1
console timeout 0
threat-detection basic-threat
threat-detection statistics access-list
no threat-detection statistics tcp-intercept
dynamic-access-policy-record DfltAccessPolicy!
class-map inspection_default
 match default-inspection-traffic
!
!
policy-map type inspect dns preset_dns_map
 parameters  
  message-length maximum client auto
  message-length maximum 512
#Some lines are commented out to disable IDS inspection of that protocol. Add or remove those protocols that should be inspected as needed
policy-map global_policy 
 class inspection_default
  inspect dns preset_dns_map
  inspect ftp
  inspect h323 h225
  inspect h323 ras
  inspect rsh
  inspect rtsp
  inspect esmtp
  inspect sqlnet
  #inspect skinny
  #inspect sunrpc
  inspect xdmcp
  inspect sip
  #inspect netbios
  inspect tftp
  inspect ip-options
!
service-policy global_policy global
prompt hostname context
```
