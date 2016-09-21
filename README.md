#pentmenu


**A bash script inspired by pentbox.**

Designed to be a simple way to implement various network pentesting functions, including network attacks, using wherever possible readily available software commonly installed on most linux distributions without having to resort to multiple specialist tools.


**Sudo is implemented where necesssary.**

Tested on Debian and Arch.

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

Alternatively, use git clone, or download the latest release from https://github.com/GinjaChris/pentmenu/releases, extract it and run the script.

## Script usage (aka how not to be a n00b; inspired by a Youtube video)

Some people just don't get it....don't be one of them!

Script usage is simple, menus are in the following format:

1) Option 1

2) Option 2

If you wish to select Option 1, press '1'.  If you wish to select option 2, press '2'.
Menus are used to select options and thus allows you to run modules.  Modules allow you to carry out actions.
Once within a module you may be asked questions.  Consider the following example:

Would you like to do something?  [y]es or [n]o (default):


If you want to do something, type 'y' and hit Return.
If you do not want to do something, type 'n' and hit Return.
If you type nothing and hit Return, the default action is carried out.  In this example, this would be the same as typing 'n' and hitting Return.

If you have understood the above, congratulations!  You are capable of using pentmenu!

## Module detail

**RECON MODULES**

* Show IP - uses curl to perform a lookup of your external IP. Runs ip a or ifconfig (as appropriate) to show local interface IP's.


* DNS Recon - passive recon, performs a DNS lookup (forward or reverse as appropriate for target input) and a whois lookup of the target.  If whois is not available it will perform a lookup against ipinfo.io (only works for IP's, not hostnames).


* Ping Sweep - uses nmap to perform an ICMP echo (ping) against the target host or network.


* Network Recon - uses nmap to identify live hosts, open ports, attempts OS identification, grabs banners/identifies running software version and attempts OS detection.  Nmap will not perform a ping sweep prior as part of this scan.  Nmap's default User-Agent string is changed to that of IE11 in this mode, to help avoid detection via HTTP. This scan can take a long time to finish, please be patient.


* Stealth Scan - TCP Port scanner using nmap to scan for open ports using TCP SYN scan.  Nmap will not perform a ping sweep prior to performing the TCP SYN scan. This scan can take a long time to finish, please be patient.


* UDP scan - uses nmap to scan for open UDP ports.


* Check Server Uptime - estimates the uptime of the target by querying an open TCP port with hping. Accuracy of the results varies from one machine to another; this does not work against all servers.


**DOS MODULES**

* TCP Syn Flood - sends a flood of TCP SYN packets using hping3.  If hping3 is not found, it attempts to use the nmap-nping utility instead. Hping3 is preferred since it sends packets as fast as possible.  Options are provided to use a source IP of your interface, or specify (spoof) a source IP, or spoof a random source IP for each packet.
Optionally, you can add data to the SYN packet.  All SYN packets have the fragmentation bit set and use hpings virtual MTU of 16 bytes, guaranteeing fragmentation.
Falling back to nmap-nping means sending X number of packets per second until Y number of packets is sent and only allows the use of interface IP or a specified (spoofed) source IP.  
A TCP SYN flood is unlikely to break a server, but is a good way to test switch/router/firewall infrastructure and state tables.
Note that whilst hping will report the outbound interface and IP which might make you think script does not work as expected, the source IP *will* be set as specified; review a packet capture of the traffic if in doubt!


* UDP Flood - much like the TCP SYN Flood but instead sends UDP packets to the specified host:port. Like the TCP SYN Flood function, hping3 is used but if it is not found, it attempts to use nmap-nping instead.  All options are the same as TCP SYN Flood, except you can specify data to send in the UDP packets.
Again, this is a good way to check switch/router throughput or to test VOIP systems.


* SSL DOS - uses OpenSSL to attempt to DOS a target host:port.  It does this by opening many connections and causing the server to make expensive handshake calculations.  This is not a pretty or elegant piece of code, do not expect it to stop immediately upon pressing 'Ctrl c', but it can be brutally effective.  
The option for client renegotiation is given;  if the target server supports client initiated renegotiation, this option should be chosen.
Even if the target server does not support client renegotiation (for example CVE-2011-1473), it is still possible to impact/DOS the server with this attack.  
It is very useful to run this against loadbalancers/proxies/SSL-enabled servers (not just HTTPS!) to see how they cope under the strain.


* Slowloris - uses netcat to slowly send HTTP Headers to the target host:port with the intention of starving it of resources.  This is effective against many, although not all, HTTP servers, provided the connections can be held open for long enough.  Therefore this attack is only effective if the server does not limit the time available to send a complete HTTP request.
Some implementations of this attack use clearly identifiable headers which is not the case here.  The number of connections to open to the target is configurable. 
The interval between sending each header line is configurable, with the default being a random value between 5 and 15 seconds. The idea is to send headers slowly, but not so slow that the servers idle timeout closes the connection.
The option to use SSL (SSL/TLS) is given, which requires stunnel and allows the attack to be used against a HTTPS server.

Defences against this attack include (but are not limited to):

Limiting the number of TCP connections per client; this will prevent a single machine from making the server unavailable, but is not effective if say, 10,000 clients launch the attack simultaneously.  Additionally, such a defensive measure may negatively impact multiple (legitimate) clients operating behind a forward proxy server.

Limiting the time available to send a complete HTTP request; this is effective since the attack relies on slowly sending headers to the server (the server should await all headers from the client before responding).  If the server limits the time for receiving all headers of a request to 10 seconds (for example) it will severely limit the effectiveness of the attack.  It is possible that such a measure will prevent legitimate clients over slow/lossy connections from accessing the site.


* Distraction Scan - this is not really a DOS attack but simply launches multiple TCP SYN scans, using hping, from a spoofed IP of your choosing (such as the IP of your worst enemy). It is designed to be an obvious scan in order to trigger any lDS/IPS the target may have and so hopefully obscure any actual scan or other action that you may be carrying out.


**EXTRACTION MODULES**

* File extraction via ICMP - This module uses hping to send data with ICMP packets.  It can be extremely useful where only ICMP connectivity is possible.


* File receipt via ICMP - This module uses hping to listen for ICMP packets and record the data to an output file of your choice. It will only record packet data starting with the secret that you define.  Therefore the extractor and receiver must use an identical secret. 

An alternative to using this receiver is to run wireshark to capture the inbound icmp packets, which seems quite happy to reconstruct the data received over several fragmented ICMP packets.


* Listener - uses netcat to open a listener on a configurable TCP or UDP port.  This can be useful for testing syslog connectivity, receive files or checking for active scanning on the network. Anything received by the listener is written out to ./pentmenu.listener.out or a file of your choice.


## Disclaimer

This script is only for responsible, authorised use. You are responsible for your own actions and this script is provided without warranty or guarantee of any kind.  The author(s) accept no responsibility or liability on your behalf.


## Also see
Pentmenu is available as a [package](https://archstrike.org/packages/pentmenu) on Arch Linux. Big love to [ArchStrike](https://archstrike.org/) and [Parrot linux](https://www.parrotsec.org/).
