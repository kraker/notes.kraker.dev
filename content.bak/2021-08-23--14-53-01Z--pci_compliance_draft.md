---
title: PCI Compliance Draft 
date: 2021-08-23 14:53 
---

Draft.

# Overview

**Payment Card Industry Data Security Standard (PCI DSS)** scans are server
checks that security assessors perform to determine if a website and server are
in compliance. Often customers will conduct a _PCI compliance scan_ for legal
reasons for sites that collect and retain credit card information. PCI Standards
are mandated by credit card companies and were created to increase controls
around card-holder data and reduce credit card fraud. 

See [PCI DSS](https://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard) 
for further information.

This guide will show the most common reports from PCI scans as well as how to
fix them (the wording will vary depending on the scan company).  Please note
that most of these changes can only be done on a VPS or dedicated server, and
the ones that can be implemented on a shared server on a per-account basis are
noted.

Notes:
* Some vulnerabilities may show up multiple times, but for different ports.
	Usually the scan report will indicate which service and/or port(s) it is
	being detected on.
* Pay attention to the 'urgency' of the reported items.  Vulnerabilities
	usually are reported on a scale of 1-5, with 5 being critical and 1 being
	informational, or a scale of High, Medium, and Low. "Failures" are the only
	issues that have to be correct in order for a customer to pass the scan.
* The customer MUST have at least one valid SSL certificate to even qualify
	for the scan, whether it be a shared one or a dedicated one.  It cannot be
	self-signed, or issued by cPanel.

# Weak Encryption, SSL/TLS Protocol and Ciphers

Weak SSL/TLS protocols and ciphers are the most common reason for PCI failures.
Fortunately, recent versions of cPanel have made it fairly straightforward to
update the SSL protocols and ciphers for various services (Apache, cPanel, exim,
dovecot, FTP). They have also updated the default ciphers on recent cPanel
versions; the default configuration of a cPanel server is fairly PCI compliant
out of the box. But it's helpful to understand how to properly configure the
appropriate protocols and cipher suites if a compliance scan fails or identifies
areas of vulnerability. 

While it may seem similar, the SSL Cipher Suite and SSL/TLS Protocols are two
separate things. One dictates what encryption and key exchange mechanisms are
used between the server and the client, and one sets the allowed connection
version or _protocol_. Also, use of similar keywords in the cipher suite and
protocols can lead to confusion on what protocols and ciphers are being used. For
example, the highest supported TLS version is 1.0 on CentOS 5. As TLS 1.0 in
itself does not specify new ciphers, if one were to disable all SSLv3 ciphers
using -SSLv3 keyword, you would end up with no usable ciphers, but it is still
possible to disable SSLv3 protocol but allow TLSv1 on CentOS 5.

When setting the cipher list, it is recommended to use Ephemeral Elliptical
Curve Diffie-Hellman (EECDH) or Ephemeral Diffie-Hellman (EDH) for the key
exchange. These provide [forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy), 
which means that in case the private key of the certificate is compromised, it
would not allow past encrypted data to be decrypted. If this prevents successful
connections, the _HIGH_ keyword should provide sufficient list of secure ciphers.
But this is dependent on the OpenSSL version installed and on how it was compiled.

You will generally see two types of failures regarding SSL/TLS:

1. SSL Certificate Signed Using Weak Hashing Algorithm (Known CA)
2. Sweet32: Birthday attacks on 64-bit block ciphers in TLS

Both of these vulerabilities can be fixed by updating the ciphers in WHM for the
affected services. 

## Fixing Cipher and Protocol Vulnerabilities

### Apache (ports 80, 443, 8080 and 8443):

In _WHM >> Service Configuration >> Apache Configuration >> Global
Configuration_.
Copy the following into **SSL Cipher Suite:**
```
ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA:!3DES:!ECDH
```
Copy the following into **SSL/TLS Protocols**
```
All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
```
Click **Save** at the bottom of the page

### Mail (ports 25, 110, 143, 465 and 587)

1. Log into WHM
* Navigate to _Service Configuration -\> Mailserver Configuration_
* Allow Plaintext Authentication (from remote clients) -\> Select "No" from
	the drop down
* Copy the following into _SSL Cipher List:_
	- <pre>ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA:!3DES:!ECDH</pre>
- Copy the following into SSL Protocols
	- `!SSLv2 !SSLv3 !TLSv1`
- Scroll down to the bottom and click **Save Changes**.

### cPanel (Ports 2083, 2087 and 2096)

* Log into WHM
* Navigate to _Service Configuration -\> cPanel Web Services Configuration_
* Copy the following into _TLS/SSL Cipher List:_
	- <pre>ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA:!3DES:!ECDH</pre>
* Copy the following into TLS/SSL Protocols
	- `SSLv23:!SSLv2:!SSLv3:!TLSv1`
* Click **Save** at the bottom of the page

### FTP(s) (Ports 21 and 22)

* Log into WHM
* Navigate to _Service Configuration -\> FTP Server Configuration_
* Change the dropdown at the top for _TLS Encryption Support_ to **Required**
* Copy the following into _TLS Cipher Suite:_
	* `HIGH:MEDIUM:+TLSv1:!SSLv2:+SSLv3`
* Click **Save** at the bottom of the page

## Checking Your Work

If you plan on working PCI compliance regularly, it is strongly suggested that
install _nmap_ on your server so you can make sure the ciphers have been fixed
after you have made the changes in WHM:

```
yum install nmap
```

Once you have _nmap_ installed, you can test the ciphers. Here is an example of a
test on port 443 for vps33417:

```
nmap --script ssl-enum-ciphers -p 443 -Pn "vps33417.inmotionhosting.com"
```

If everything is done correctly, you should see something like this at the
bottom of the output:

```
|     cipher preference: client
|_  least strength: A`
```

**SIDE NOTES** A few notes on what is excluded, and why. \!NULL prevents ciphers
without encryption (such thing exists). \!aNULL on the other hands prevents use
of anonymous DH/ECDH key exchanges, which are vulnerable to man-in-the-middle
attack. \!DSS excludes rarely used DSS certificates, \!SRK and \!PSK excludes
pre-shared keys ciphers, and \!RC4 and \!DES prevents weak encryption. Finally,
@STRENGTH sorts it by what OpenSSL believes to be the strongest ciphers first.
You can verify what ciphers will be used by using the following command:

```
openssl ciphers -v
```

Pay particular attention to Au=None (No authenticated key exchange\!), Enc=None
(No encryption\!) or Enc=xxx(112) or lower (weak encryption).  If any of those
show up, you will need to exclude it. The same cipher list can be used for any
of the services as needed.

At this time SSLv3 is considered broken, and TLSv1 should be the minimum being
used. Because it can cause issues in the future, the use of .local file to make
configuration changes is discouraged. While all the services can be configured
through WHM, each one has a different syntax:

```
Apache: All -SSLv2 -SSLv3
cPanel: SSLv23:!SSLv2:!SSLv3
Dovecot: !SSLv2 !SSLv3
exim: +no_sslv2 +no_sslv3
```

Note that SSLv23 for protocols is equivalent to All, and does not mean "use only
SSLv2 or SSLv3". For Apache, it's recommended to use SSLHonorCipherOrder On to
make sure the strongest cipher is being used first, which needs to be added to
the Pre Main Includes.

# Other Commonly Failed Tests

## Open Ports / Firewall Warning

Some scans will alert about different ports being opened and will want you to
make sure that any unnecessary services are disabled and ports closed in the
firewall. Some will even determine that if multiple ports are indeed open, that
the server does not have a firewall. In some cases, the scan company will want a
list of open ports and what they are being used for, which you can provide them
(as they already know the open ports by doing a port scan)

The most commonly flagged ports that you may need to close for scans are going
to be 110 (mail), 143 (mail), 587 (mail), 2082 (cPanel), 2086 (WHM), 2095
(Webmail) and 3306 (database/sql).

These ports accept traffic/passwords over unencrypted channels and should be
closed for compliance. Before closing them, it is important that you advise the
customer of the consequences of closing the port.  For instance, closing the
mail ports will require that anyone connecting to the customer's server for
email will need to change their email client settings from non-SSL to the SSL
settings. Closing the unencrypted cPanel, WHM and webmail ports will require
that the customer connects to these services via the encrypted ports instead
(2083, 2087 and 2086, respectively).

To close the ports, open the APF conf file at /etc/apf/conf.apf with your
favorite text editor. The section that you need to modify fill look like this:

```
# Common inbound (ingress) TCP ports
IG_TCP_CPORTS="20,21,25,53,80,110,143,443,465,587,993,995,2079,2080,2082,2083,2086,2087,2095,2096,3306,30000_35000"
```

Remove the ports from that list that you need to close, and save the file. Then
you need to restart APF:

```
apf -r
```

There will be some cases when the report will generate a false positive on a
port that needs to be open in order for a service to run. While these cases are
rare, you might see them have an issue with one of these ports, such as port 80.

**Here is a good canned response to this vulnerability:**

> The warning regarding "open ports" on your server does not necessarily indicate
> an actual security risk. Each of these ports is valid, and are required for the
> proper functionality and maintenance of your website, e-mail, and other
> services. There is, in fact, a firewall in place on the server, and we actively
> block all ports that are actually insecure.  Disabling or blocking these ports
> is not recommended, and you can inform your scanning company that your server
> runs cPanel, and that all of the open ports listed in their report are actually
> safe and secured. Note that if you have your own V/Dedicated solution, we can
> block ports at your request in your server's firewall.

## Outdated System Software 

_(Not to be confused with Vulnerabilities - we will discuss that below)_

In many cases, Outdated System Software failures are caused by austomer running
Easy Apache 3 on an older node running CentOS 5. You can check with the
following:

```
cat /etc/redhat-release
```

If the output shows that they are on CentOS 5, we will need to make a ticket to
T2S to have their container migrated to a newer node with updated software.
However, we must review the implications of the changes that will be made to
their server before we migrate them. You can use the following canned answer to
confirm their approval before the move:
 
> It appears that you failed your PCI compliance test due to Outdated System 
> Software. In order to correct this, we will need to migrate your VPS to a newer 
> node that is running the latest available version of CentOS. Before we do that, 
> I wanted to go over a few changes that will take place:
> 
> * The main IP address and hostname of their server will change. Any domains 
> 	pointing to the main IP will undergo a period of propagation while the DNS 
> 	information updates to point to the new server which may take 4-24 hours.
> * All dedicated IPs will be migrated over to the new server.
> * The glue records for your custom nameservers will need to be updated to match 
> 	the IPs on the new server.
> * The main PHP and MySQL versions the websites run on will be updated. Please 
> 	confirm with your developer that there would not be an issue with newer versions 
> 	of PHP and MySQL.
> * Any modifications done to the server with root access or requested custom 
> 	configurations will not carry over to the new server.
> * PHP 5.2 will no longer be available.
> 
> Please review the above changes, and let us know if it is OK to proceed. Once we 
> have your confirmation, we will be happy to assign this ticket to our Systems 
> Team to have your VPS migrated.

### Apache/PHP

If the customer is on a VPS or Dedicated server, you can upgrade their PHP and
Apache versions using the standard upgrade procedure.

[EA4 Migrations](http://wiki.inmotionhosting.com/index.php?title=EasyApache_4)

### Vulnerable Versions of Pre-Installed Software

Most Scanning Companies will not properly recognize backports, likely because
they don't use the proper Nessus plugins for it. If the customer is on CentOS 6
or higher, and the report marks vulnerabilities for versions of services like
Exim, BIND, OpenSSL, etc., it is likely that the vulnerabilities have been
backported. RHEL based operating systems use backporting instead of updates, as
it's a more stable approach to fixing vulnerabilites while ensuring the serivce
still works with everything else on the server.

Let's say for instance, that you get a report that Exim is vulnerable to
CVE-2017-16944, which caused the test to fail. You will want to check the
changelog on the server to ensure that Exim has been backported against the
vulnerability. Here is how you can do that:

```
CT-20331-bash-4.1# rpm -q --changelog "exim" | grep 16943
- Applied patch for CVE-2017-16943 and CVE-2017-16944
CT-20331-bash-4.1#
```

This indicates that the vulnerability has been backported, and is not a factor
in their sever's overall security. Here's an example of an answer that you can
send to a customer in that situation:

> With respect to the Exim Vulnerability, your server is running CentOS which is
> a Redhat based operating system. These operating systems backport the Exim
> version instead of upgrading it because it is a much more stable approach to
> addressing vulnerabilities while ensuring that the software is compatible with
> other functions on the server. Back porting is a method of patching software to
> fix vulnerabilities rather than re-coding it into a whole new version.
> Essentially this means that the report that you have provided is generating
> false positives, and your server is protected from the Exim Vulnerability. This
> can be seen in the changelog on the server:
> 
> <pre>root@server [/]# exim --version
> Exim version 4.94.2 #2 built 07-May-2021 10:34:38
> Copyright (c) University of Cambridge, 1995 - 2018
> (c) The Exim Maintainers and contributors in ACKNOWLEDGMENTS file, 2007 - 2018
> Berkeley DB: Berkeley DB 5.3.21: (May 11, 2012)
> Support for: crypteq iconv() IPv6 PAM Perl OpenSSL Content_Scanning DANE DKIM DNSSEC Event I18N OCSP PIPE_CONNECT PRDR SPF Experimental_SRS
> Lookups (built-in): lsearch wildlsearch nwildlsearch iplsearch cdb dbm dbmjz dbmnz dnsdb dsearch passwd sqlite
> Authenticators: cram_md5 dovecot plaintext spa
> Routers: accept dnslookup ipliteral manualroute queryprogram redirect
> Transports: appendfile/maildir autoreply lmtp pipe smtp
> Malware: f-protd f-prot6d drweb fsecure sophie clamd avast sock cmdline
> Configure owner: 0:0
> Size of off_t: 8
> 2021-08-23 15:15:44.438 [17950] cwd=/ 2 args: exim --version
> Configuration file is /etc/exim.conf</pre>
> 
> <pre>root@server [/]# rpm -q --changelog "exim" | grep 6789
> - Fix CVE-2018-6789</pre>
> 
> You should be able to contact the company running this scan to report the false
> positive by providing the information mentioned in this email.  The company
> should be able to confirm that the vulnerability has been patched, and then
> exclude it from the results of the scan.

### Mod\_Frontpage

**Examples:**

```
mod_frontpage older than 1.6.1 is vulnerable to a buffer overflow which may allow an attacker to gain root access.
```

This is true on some servers, but on newer systems (mostly compiled after 8/07)
this is a false-positive as cPanel has backported the frontpage module and
though it reads 1.6.1, it is actually 1.6.2. The customer can report this as a
false-positive, but if they are on a VPS/dedication and are not using Frontpage,
you can simply remove the mod\_frontpage loader from httpd.conf.

## Anything about port 8443

**Examples:**

```
The remote service supports the use of weak SSL ciphers. Description :
```

To block access to port 8443 on a VPS, run the following from the node:

```
vzctl set VEID --offline_management no --save
```

## cPanel guestbook.cgi is present

**Examples:**

<http://domain.com/cgi-sys/guestbook.cgi>

You can disable this by making brief changes to the VirtualHost for the domain
in question. This should be done with include files and not directly in the
httpd.conf.

```
Uncomment (Include "/usr/local/apache/conf/userdata/std/2/userna5/domain.com/*.conf") in the httpd.conf for the specific domain.
mkdir -p /usr/local/apache/conf/userdata/std/2/userna5/domain.com/
echo "ScriptAlias /cgi-sys/ /home/userna5/public_html/cgi-bin/" > /usr/local/apache/conf/userdata/std/2/userna5/domain.com/disable_cgisys.conf
/scripts/rebuildhttpdconf
service httpd restart
```

After making the above change, it is likely the guestbook will still be
accessible via https. You can link the disable\_cgisys.conf script to the SSL
includes of the virtual host by making a softlink similar to the following.
Notice the "std" vs "ssl" directory.

```
ln -s /usr/local/apache/conf/userdata/std/2/userna5/domain.com/disable_cgisys.conf /usr/local/apache/conf/userdata/ssl/2/userna5/domain.com/disable_cgisys.conf
```

## Clear Text Email Connections

On newer versions of cPanel, the fix listed below will work for a day or so and
then revert back. To make it permanent, you need to edit the setting in
_WHM -\> Exim Configuration Manager -\> Security -\> 
Require clients to connect with SSL or issue the STARTTLS command before they are allowed to authenticate with the server. [Enabled]_

To test this, you can telnet to the server as below:

```
telnet [server] 25
ehlo
```

You should get this:

```
250-SIZE 52428800  
250-8BITMIME 
250-PIPELINING
250-STARTTLS <- STARTTLS is enabled/enforced  
250 HELP
```

## Dovecot

In `/etc/exim.conf` locate:

```
begin authenticators
  
fixed_plain:
  driver = plaintext  
  public_name = PLAIN
  server_prompts = :  
  server_condition = "${perl{checkuserpass}{$1}{$2}{$3}}"
  server_set_id = $2
```

On newer servers with newer versions of Exim, the section may show as the
following:

```
begin authenticators  
  
  driver = plaintext
  public_name = PLAIN
  server_prompts = :
  server_condition = ${if and{{!match {$auth2}{\N[/]\N}}{eq{${if match {$auth2}{\N[+%:@]\N}{${lookup{${extract{2}{+%:@}{$auth2}}}lsearch{/etc/demodomains}{yes}}}{${loo$
  server_set_id = $auth2
```

For fixed\_plain/courier\_plain and the section below it, change the
public\_name to DIGEST-MD5 for the first one and MD5 for the second -- we'll
enable CRAM-md5 next, but this is just changing how other servers see the
plaintext authentication (in other words, no change is being made to the mail
system we're just tricking the scanner).

At the top of the file, add these lines:

```
AUTH_CRAM_MD5=yes
AUTH_PLAINTEXT=no 
AUTH_SPA=yes
```

Restart Exim then run the following to make sure your settings are working as
intended.

```
telnet $server 25
```

Then type _ehlo_, it should look something like this:

```
[anthonye@work ~]$ telnet vps#### 25 Trying 205.134.239.51...  
Connected to vps####. 
Escape character is '^]'.  
ehlo
220-vps####.inmotionhosting.com ESMTP Exim 4.80.1 #2 Fri, 01 Nov 2013 08:01:25 -0700 
220-We do not authorize the use of this system to transport unsolicited, 220 and/or bulk e-mail.  
250-vps####.inmotionhosting.com Hello vbo5.inmotionhosting.com [216.54.31.86] 
250-SIZE 52428800 250-8BITMIME 250-PIPELINING 250-AUTH DIGEST-MD5 MD5 
250-STARTTLS 
250 HELP
```

If you see `250-AUTH PLAIN LOGIN` then please review the `/etc/exim.conf` edits.

## Unix Account Names

**Examples:**

```
Discovery of Unix Account Names Vulnerability
```

The scan is basically just asking you to remove userdir from the server.  You
can disable this on VPS and dedicated servers via WebHost manager completely,
and per-account on shared servers.

Note that some servers have the userdir setting switched, and when it says that
it's off it's usually on and vice-versa. Make sure to test the temp URL before
and after disabling userdir to make sure it is in fact disabled. If userdir is
successfully disabled for a user you will get a 404 error when accessing their
temp URL.

**To disable per user**, edit `/var/cpanel/templates/apache2/main.local` and
locate:

```
Userdir public_html
```

And below it add:

```
Userdir disabled root
```

The rebuild `httpd.conf` and restart Apache. If main.local does not exist, simply
copy it from main.default.

**To disable all together (VPS and DEDI ONLY):**  UserDir disabled

## Remote Web Server Type

**Examples:**

```
The remote web server type is : Apache/1.3.37 (Unix) etc.....
```
```
HTTP Server type and version
```
```
Linux Distribution Detection
```

The scan is basically complaining that Apache is sending out headers that give
too much information on the server. On VPS and dedicated servers you can
add/edit the _ServerSignature_ and _ServerToken_ lines (depending on what the scan
is asking for):

```
ServerSignature Off
ServerTokens Prod
```

Information for these directives can be found
[here](http://www.petefreitag.com/item/419.cfm)

This will work on both Apache 1.3.x and Apache 2.x

## Apache HTTP Server 413

**Examples:**

```
Apache HTTP Server 413 Error HTTP Request Method Cross-Site Scripting Weakness
```
```
Apache HTTP servers are prone to a cross-site scripting weakness.
```

There is no known 'fix' for this, but you can work around it by adding an
ErrorDocument directive to httpd.conf (VPS/dedicated) or to .htaccess (shared
servers):

```
ErrorDocument 413 /index.php  (or some other file)
```

With that said, the customer really needs to work with their developer to
correct the problem as it's coding related.

## Trace/Track Methods

**Examples:**

```
Web Server HTTP Trace/Track Method Support Cross-Site Tracing Vulnerability
```

The best way to disable this is to go into WHM \>Apache configuration \>Global
configuration and set TraceEnable to off.

The customer can add these lines to their .htaccess:

```
RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK)$ [NC]`  `RewriteRule ^.*$ - [F]
```

If the scan still reports this, add to httpd.conf in their VirtualHost entry.

## Directory Indexes

**Examples:**

```
Web Directories Listable Vulnerability
```

The report will usually list the directories that fail this test, but in general
the customer should disable directory indexes from their cPanel, or insert blank
index pages into the directories that the scan lists.

## Mailman

cPanel Official Documentation on disabling **Mailman** can be found
[here](https://docs.cpanel.net/knowledge-base/security/pci-compliance-and-software-versions/#mailman). 

_Information in this section below this has been deprecated. It's candidate for
removal from this wiki. - Alex Kr_
PCI Scans now check for /mailman, which is a valid alias in the Apache config.
Verify mailman admin access by visiting:

<http://cxdomain.com/mailman/admin>

In `/var/cpanel/conf/apache/main`, remove the following lines:

```
- 
	path: /usr/local/cpanel/3rdparty/mailman/archives/public/ 
	url: /mailman/archives
```

and

``` 
- 
	path: /usr/local/cpanel/3rdparty/mailman/cgi-bin/ 
	url: /mailman 
```

Rebuild the apache conf, and restart. Simply deleting the mailman folders on the
server is a temporary fix - a cpanel update will put them right back.

Verify fix by visting cx site again:

<http://cxdomain.com/mailman/admin>

## DNS Cache Snooping for named / BIND

By default, BIND will allow recursive queries for lookups on other domains that
are not master zones on the name server. This presents some PCI compliance
issues and some informational vulnerabilities (allowing third parties to query
the nameserver). It is important to restrict who can perform DNS queries, in
addition to what is allowed to be queried.  If this DNS server is only meant to
be recursively queried by internal users for third-party domains, then there is
no reason to allow the general internet to also perform queries against it. If
the server is meant only to act as a nameserver for specific domains, then
recursive queries should be disabled as it is unnecessary for the server to
resolve anything other than its own domains.

- To disable recursive queries, add the following to the **options** section
	of **named.conf**:

```
allow-transfer {"none";};
allow-recursion {"none";};
recursion no;
```

Then restart the named service and dig at the name server to ensure the changes
have taken effect. You can do so by [querying the server for a DNS entry for
which it is not authoritative](http://serverfault.com/q/371774/59321), such as
google.com:

```
host google.com vps3438.inmotionhosting.com`
```

## scgi-bin Issues

PCI compliance companies may fail a scan due to the way that
`/usr/local/cpanel/cgi-sys/scgiwrap` works:

This script is used to run cgi scripts as the cPanel user instead of the
'nobody' user. But what ends up happening is that PCI companies will slam the
script with requests to files that aren't present. Instead of giving an HTTP 404
message, the script will return HTTP 200 OK, and the scanner used will mark this
as a present script.

In order to disable this functionality, edit /var/cpanel/conf/apache/main and
near line 266, remove the following lines:

```
- 
	path: /usr/local/cpanel/cgi-sys/scgiwrap` 
	url: /scgi-bin`
```

Then, rebuild the Apache configuration and restart Apache:

```
/scripts/rebuildhttpdconf 
service httpd graceful
```

## Clickjacking Issues

You can add this to the `post_virtualhost_global.conf`

```
Header always append X-Frame-Options SAMEORIGIN
```

Check the headers using a tool like
[this](http://tools.geekflare.com/seo/tool.php?id=check-headers) and look for
the X-Frame-Options: SAMEORIGIN line.

For Clickjacking issues on cPanel ports (2083,2096,etc.), the option to add
X-Frame-Options can be enabled in WHM \> Tweak Settings \> "Use X-Frame-Options
and X-Content-Type-Options headers with cpsrvd".

Or whmapi can be used to enable like below:

```
whmapi1 set_tweaksetting key=xframecpsrvd value=1
```

# Informational Reports (no action needed)

This section includes typical level 1 and 2 vulnerabilities that are note
required for a host to pass a scan, but that customers may want to address
anyways.

## Path Disclosure

**Examples:**

```
The Web site appears to display paths of local directories.
```

This is usually reported as informational and not a critical issue. For paths
that should not be displaying, you can deny them via a Location tag in
httpd.conf:

```
<Location "/usr/local/apache/conf/*">
	AllowOverride None
	deny from all  
</Location>
```

## PHP Information Disclosure

**Examples:**

```
'phpinfo.php' Information Disclosure
```

You can remove the phpinfo.php files from the server, or add 'phpinfo' to the
list of disable\_functions in php.ini.

Apparently some ASVs think the PHP credits page discloses too much info, so you
can set "expose\_php = Off" in the php.ini to disable the credits page, as well.

## Global User List

**Examples:**

```
This is the global system user list, which was retrieved during the scan by 
exploiting one or more vulnerabilities.
```

The scan will usually mention 'root' and 'operator' in the list of usernames.
You can remove operator by using the 'userdel' command.  Usually though this is
not an issue.

## Expose\_php Set to On in php.ini

**Examples:**

```
Expose_php Set to On in php.ini
```

You can disable this in php.ini by setting 'expose\_php' to 'Off'. This cannot
be disabled via .htaccess.

## Apache eTAG Header

**Examples:**

```
Apache Web Server ETag Header Information Disclosure Weakness
```

The eTag displays inode information and other non-critical server info.  You can
disable this in httpd.conf (VPS/dedicated) by adding this line:

```
FileETag None
```

On shared servers you would need to add a directory tag in the customer's vhost
entry (will not work in .htaccess):

```
<Directory /home/user/public_html>  
	FileETag None
</Directory>
```

## printev

**Examples:**

```
'printenv' Information Disclosure
```

Delete /usr/local/apache/htdocs/cgi-bin/printenv

## named / BIND Hostname Disclosure

By default, bind will alow queries of the server hostname as reported in the DNS
configuration. This will sometimes show up as a "Low" vulnerability in scans but
be enough to fail it with some compliance companies. You can check to see if it
is enabled by doing the following

```
dig hostname.bind TXT CHAOS @IP.ADDRESS.OF.SERVER +short
```

This functionality can be disabled by adding the following to the
`/etc/named.conf` _options_ section:

```
hostname none;
```

Restart bind/named and run the test above again. If nothing is returned then it
is properly disabled.

# Website-level Reports

PCI scans will frequently report issues with the software on a customer's site,
such as XSS problems or known vulnerable scripts. They usually appear as
detailed reports of request types or files and show up on port 80 and/or 443. As
long as they are not related to one of cPanel's functions (such as the cgi-sys
or scgi-bin folder above), these issues are really up to the customer or
customer's developer to address, as most can usually be fixed by a software
update or configuration change.

Customers who have websites built by our design team usually have their sites
scanned by Trustwave, which is fairly permissive and any software-level errors
can usually be addressed by a simple update from IMW (if they show up at all\_.
In the case that the customer uses another scanning company or runs into
undiscovered issues, they can be handled on a case-by-case basis to see if we
can implement any easy fixes (alternate configuration, mod\_security rules, and
such). Any fixes can be documented below for future use:

## User specified URL redirection (Open Redirect)

For OpenCart users who are being scanned by McAfee, you may get a warning about
a "User specified URL redirection (Open Redirect)". This is an issue with the
way OpenCart handles information sent via POST data to the "redirect" variable,
which is used by some pages to direct the customer to a new location after an
action is taken (such as a checkout sending them back to the homepage). It has
been covered in the OpenCart forum before, and several instances of it have been
patched (see [this forum
topic](http://forum.opencart.com/viewtopic.php?f=10&t=12043&start=20) for some
additional information). For a newer version of OpenCart, you can implement the
following mod\_security rule to prevent any redirections to sites other than the
customer's:

```
SecRequestBodyAccess On
SecRule ARGS:redirect !^(http\:\/\/DOMAIN.COM.*|http\:\/\/www.DOMAIN.COM.*)$ "phase:2,drop,log,status:406,msg:'URL Redirection Exploit',severity:'2'"
```

Make sure to replace DOMAIN.COM with the base domain/subdomain for the store,
adjusting the second case for **www.** accordingly, or removing it entirely. You
can add those rules into normal VHost include files; simply create the following
directories if they don't already exist:

```
mkdir -p /usr/local/apache/conf/userdata/std/2/USERNA5/DOMAIN.COM
mkdir -p /usr/local/apache/conf/userdata/ssl/2/USERNA5/DOMAIN.COM
```

Add a file named modsec-pci.conf to each folder, containing the two lines above;
rebuildhttpdconf and restart Apache; and you should be good to go. Note that
this is only written for servers using mod\_security 2; anything with an older
version will likely not be compatible with a new copy of OpenCart.

# Physical location questions

Information on our physical Data Centers with regards to PCI questions can be
found here: [SAS\_70](SAS_70 "wikilink")

# 3rd Party Tools

* [testssl.sh](https://github.com/drwetter/testssl.sh)

Running cPanel **System Status Probe** may also reveal certain system
vulnerabilities. This can be run with:
```
/usr/local/cpanel/3rdparty/bin/perl <(GET https://ssp.cpanel.net/ssp)
```
_Note: This 3rd party script is a T3 approved script._

# See also

* <http://twiki.cpanel.net/twiki/pub/AllDocumentation/TrainingResources/TrainingSlides07/PCI_Compliance.pdf>
* <http://www.docs.cpanel.net/twiki/bin/view/AllDocumentation/PCIComplianceInfo/WebHome>
* <http://www.v-nessa.net/2008/04/14/moving-towards-pci-compliance-with-cpanel>
