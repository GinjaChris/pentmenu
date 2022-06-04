# pentmenu

[中文说明](/README.zh-CN.md)

**A bash select menu for quick and easy network recon and DOS attacks**


Sudo is implemented where necessary.
Tested on Debian and Arch.

**Contributions and pull requests are most welcome!**

## Requirements:

* bash

* sudo 

* curl

* netcat (must support '-k' option, openbsd variant recommended)

* hping3 (or nping can be used as a substitute for flood attacks)

* openssl

* stunnel

* nmap

* whois (not essential but preferred)

* nslookup (or 'host')

* ike-scan

* bind-tools (often part of the 'bind' package, needed for "host" and "nslookup")

## How to use?


- Download the script:

```
$ wget https://raw.githubusercontent.com/GinjaChris/pentmenu/master/pentmenu
```

- Make it executable:

```
$ chmod +x ./pentmenu
```

- Run it:

```
$ ./pentmenu
```

Alternatively, download the latest release from https://github.com/GinjaChris/pentmenu/releases, extract it and run the script.
Or use git clone:

```
git clone https://github.com/GinjaChris/pentmenu
```

## Module detail

**RECON MODULES**

* Show IP - uses curl to perform a lookup of your external IP. Runs ip a or ifconfig (as appropriate) to show local interface IP's.


* DNS Recon - passive recon, performs a DNS lookup (forward or reverse as appropriate for target input) and a whois lookup of the target.  If whois is not available it will perform a lookup against ipinfo.io (only works for IP's, not hostnames).


* Ping Sweep - uses nmap to perform an ICMP echo (ping) against the target host or network.


* Quick Scan - TCP Port scanner using nmap to scan for open ports using TCP SYN scan.  Nmap will not perform a ping sweep prior to performing the TCP SYN scan. This module scans the 1,000 most common ports. This module can, of course, be used to scan a single host or a full network but is really designed to identify targets across a range of IP addresses. This scan can take a long time to finish, please be patient.


* Detailed Scan - uses nmap to identify live hosts, open ports, attempts OS identification, grabs banners/identifies running software version.  Nmap will not perform a ping sweep prior as part of this scan.  Nmap's default User-Agent string is changed to that of Microsoft Edge browser in this mode, to help avoid detection via HTTP. *All* TCP ports on the target (hostname/IP/subnet) are scanned. Whilst module can scan a single host or many hosts, its intended use is to perform an information gathering scan against a single system. This scan can take a long time to finish, please be patient.


* UDP scan - uses nmap to scan for open UDP ports. *All* UDP ports are scanned.


* Check Server Uptime - estimates the uptime of the target by querying an open TCP port with hping3. Accuracy of the results varies from one machine to another; this does not work against all servers.


* IPsec Scan - attempts to identify the presence of an IPsec VPN server with the use of ike-scan and various Phase 1 proposals. Any text output from this module, whether it be regarding "handshake" or "no proposal chosen", indicates the presence of an IPsec VPN server.  See http://nta-monitor.com/wiki/index.php/Ike-scan_User_Guide for an excellent overview of ike-scan and VPN phase 1.


**DOS MODULES**

* ICMP Echo Flood - uses hping3 to launch a traditional ICMP Echo flood against the target.  On a modern system you are unlikely to achieve much, but it is useful to test against firewalls to observe their behaviour.  Use 'Ctrl C' to end the flood.
The source address of flood packets is configurable. Note that the target can be an IP (i.e 127.0.0.1) or a hostname (i.e localhost.localnet.com).  Do NOT include the protocol as part of the target (i.e. http://localhost.localnet.com).


* ICMP Blacknurse Flood - uses hping3 to launch an ICMP flood against the target.  ICMP packets are of type "Destination Unreachable, Port Unreachable". This attack can cause high CPU usage on many systems.  Use 'Ctrl C' to end the attack.  See http://blacknurse.dk/ for more information. The source address of flood packets is configurable. 


* TCP SYN Flood - sends a flood of TCP SYN packets using hping3.  If hping3 is not found, it attempts to use the nmap-nping utility instead. Hping3 is preferred since it sends packets as fast as possible.  Options are provided to use a source IP of your interface, or specify (spoof) a source IP, or spoof a random source IP for each packet.
Optionally, you can add data to the SYN packet.  All SYN packets have the fragmentation bit set and use hpings virtual MTU of 16 bytes, guaranteeing fragmentation.
Falling back to nmap-nping means sending X number of packets per second until Y number of packets is sent and only allows the use of interface IP or a specified (spoofed) source IP.  
A TCP SYN flood is unlikely to break a server, but is a good way to test switch/router/firewall infrastructure and state tables.
Note that whilst hping will report the outbound interface and IP which might make you think script does not work as expected, the source IP *will* be set as specified; review a packet capture of the traffic if in doubt!
Since the source is definable, it is simple to launch a LAND attack for example (see https://en.wikipedia.org/wiki/LAND).  The ability to set the source also allows, for example, sending SYN packets to one target and forcing the SYN-ACK responses to a second target.


* TCP ACK Flood - offers the same options as the SYN flood, but sets the ACK (Acknowledgement) TCP flag instead.
Some systems will spend excessive CPU cycles processing such packets.  If the source IP is set to that of an established connection, it is possible that an estabished connection can be disrupted by this 'blind' TCP ACK Flood.  This attack is considered 'blind' because it does not take into account any details of any established connection (like sequence or acknowledgement numbers).  


* TCP RST Flood - offers the same options as the SYN flood, but sets the RST (Reset) TCP flag instead.
Such an attack could interrupt established connections if the source IP is set to that of an established connection.
See https://en.wikipedia.org/wiki/TCP_reset_attack for example.


* TCP XMAS Flood - similar to the SYN and ACK floods, with the same options, but sends packets with all TCP flags set (CWR,ECN,URG,ACK,PSH,RST,SYN,FIN).  The packet is considered to be 'lit up like a christmase tree'.  Theoretically at least, such a packet requires more resources for the receiver to process than a standard packet.
However, such packets are quite indicative of unusual behaviour (such as an attack) and are usually easily identified by IDS/IDP.


* UDP Flood - much like the TCP SYN Flood but instead sends UDP packets to the specified host:port. Like the TCP SYN Flood function, hping3 is used but if it is not found, it attempts to use nmap-nping instead.  All options are the same as TCP SYN Flood, except you must specify data to send in the UDP packets.
Again, this is a good way to check switch/router throughput or to test VOIP systems.


* SSL DOS - uses OpenSSL to attempt to DOS a target host:port.  It does this by opening many connections and causing the server to make expensive handshake calculations.  This is not a pretty or elegant piece of code, do not expect it to stop immediately upon pressing 'Ctrl c', but it can be brutally effective.  
The option for client renegotiation is given;  if the target server supports client initiated renegotiation, this option should be chosen.
Even if the target server does not support client renegotiation (for example CVE-2011-1473), it is still possible to impact/DOS the server with this attack.  
It is very useful to run this against loadbalancers/proxies/SSL-enabled servers (not just HTTPS, but any SSL or TLS encrypted service!) to see how they cope under the strain.


* Slowloris - uses netcat to slowly send HTTP Headers to the target host:port with the intention of starving it of resources.  This is effective against many, although not all, HTTP servers, provided the connections can be held open for long enough.  Therefore this attack is only effective if the server does not limit the time available to send a complete HTTP request.
Some implementations of this attack use clearly identifiable headers which is not the case here.  The number of connections to open to the target is configurable. 
The interval between sending each header line is configurable, with the default being a random value between 5 and 15 seconds. The idea is to send headers slowly, but not so slow that the servers idle timeout closes the connection.  For example, if we send one header line every 900 seconds, the likelyhood is that the server will have closed the connection long before we send a second header line.
The option to use SSL (SSL/TLS) is given, which requires stunnel and allows the attack to be used against a HTTPS server.  You don't use the SSL option against a plain HTTP server.

Defences against this attack include (but are not limited to):

Limiting the number of TCP connections per client; this will prevent a single machine from making the server unavailable, but is not effective if say, 10,000 clients launch the attack simultaneously.  Additionally, such a defensive measure may negatively impact multiple (legitimate) clients operating behind a forward proxy server.

Limiting the time available to send a complete HTTP request; this is effective since the attack relies on slowly sending headers to the server (the server should await all headers from the client before responding).  If the server limits the time for receiving all headers of a request to 10 seconds (for example) it will severely limit the effectiveness of the attack.  It is possible that such a measure will prevent legitimate clients over slow/lossy connections from accessing the site.


* IPsec DOS - uses ike-scan to attempt to flood the specified IP with Main mode and Aggressive mode Phase 1 packets from random source IP's.  Use the IPsec Scan module to identify the presence of an IPsec VPN server.


* Distraction Scan - this is not really a DOS attack but simply launches multiple TCP SYN scans, using hping3, from a spoofed IP of your choosing (such as the IP of your worst enemy). It is designed to be an obvious scan in order to trigger any lDS/IPS the target may have and so hopefully obscure any actual scan or other action that you may be carrying out.


* DNS NXDOMAIN Flood - this attack uses the netcat and is designed to stress test your DNS server by sending a flood of DNS queries for (mostly) non-existent domains as well as some malformed DNS requests.  When run against a recursive DNS server it tries to tie up the server and fill the cache with negative responses, slowing/preventing legitimate queries.  Works best launched from multiple attacking clients. Use 'Ctrl c' to stop the attack.


**EXTRACTION MODULES**

* Send File - This module uses netcat to send data with TCP or UDP.  It can be extremely useful for extracting data.
An md5 and sha512 checksum is calculated and displayed prior to sending the file.
The file can be sent to a server of your choice; the Listener is designed to receive these files.


* Listener - uses netcat to open a listener on a configurable TCP or UDP port.  This can be useful for testing syslog connectivity, receive files or checking for active scanning on the network. Anything received by the listener is written out to ./pentmenu.listener.out or a file of your choice.
When receiving files over UDP, the listener must be manually closed with 'Ctrl C'.  This is because we have to force netcat to stay open to receive multiple packets, since UDP is a connectionless protocol.
When receiving files over TCP, the connection automatically closes after the client closes their connection (once the file is transferred) and md5 and sha512 checksums are calculated for the received file.


## Disclaimer

This script is only for responsible, authorised use. You are responsible for your own actions and this script is provided without warranty or guarantee of any kind.  The author(s) accept no responsibility or liability on your behalf.


## Also see

Pentmenu is available as a [package](https://archstrike.org/packages/pentmenu) on Arch Linux. Big love to [ArchStrike](https://archstrike.org/) and [Parrot linux](https://www.parrotsec.org/).


## Donations

Donations are accepted in cryptocurrency:

Bitcoin:
```
18N7UavMWKKa3sFD37WuMTnn6PdfZA3ips
```

