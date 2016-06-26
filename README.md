#pentmenu


A bash script inspired by pentbox.

Designed to be a simple way to implement various network pentesting functions, including network attacks, using wherever possible readily available software commonly installed on most linux distributions without having to resort to multiple specialist tools.

Currently implemented modules:

*Show IP

*Ping sweep

*Stealth Scan (TCP SYN Port scanner)

*UDP Scan

*Network Recon

*TCP Syn Flood

*UDP Flood

*SSL DOS

*Slowloris

*Create a TCP or UDP listener

*More to come...


Sudo is implemented where necesssary.

Tested on Debian and Arch.

Requirements:

bash

sudo 

curl

netcat

hping3 (or nping)

openssl

stunnel

nmap


#How to use?


- Download the script:

$wget https://raw.githubusercontent.com/GinjaChris/pentmenu/master/pentmenu

- Make it executable:

$chmod +x ./pentmenu

- Run it:

$./pentmenu

Alternatively, use git clone, or download the latest release from https://github.com/GinjaChris/pentmenu/releases, extract it and run the script.


#More detail

*RECON MODULE*


*Show IP - uses curl to perform a lookup of your external IP. Runs ip a or ifconfig (as appropriate) to show local interface IP's.


*Ping Sweep - uses nmap 


*Network Recon - uses nmap to identify live hosts, open ports, attempts OS identification, grabs banners/identifies running software version and attempts OS detection.


*Stealth Scan - TCP Port scanner using nmap to scan for open ports using TCP SYN scan.


*UDP scan - uses nmap to scan for open UDP ports.


*Listener - uses netcat to open a listener on a configurable TCP or UDP port.  This can be useful for testing syslog connectivity, ad-hoc communication, or checking for active scanning on the network.


*DOS MODULE*

*TCP Syn Flood - sends a flood of TCP SYN packets using hping3.  If hping3 is not found, it attempts to use the nmap-nping utility instead. Hping3 is preferred since it sends packets as fast as possible.  Options are provided to use a source IP of your interface, or specify (spoof) a source IP, or spoof a random source IP for each packet. 
Falling back to nmap-nping means sending X number of packets per second until Y number of packets is sent and only allows the use of interface IP or a specified (spoofed) source IP.  
A TCP SYN flood is unlikely to break a server, but is a good way to test switch/router/firewall infrastructure and state tables.  


*UDP Flood - much like the TCP SYN Flood but instead sends UDP packets to the specified host:port. Like the TCP SYN Flood function, hping3 is used but if it is not found, it attempts to use nmap-nping instead.  All options are the same as TCP SYN Flood, except you can specify data to send in the UDP packets.
Again, this is a good way to check switch/router throughput or to test VOIP systems.


*SSL DOS - uses OpenSSL to attempt to DOS a target host:port.  It does this by opening many connections and causing the server to make expensive handshake calculations.  This is not a pretty or elegant piece of code, do not expect it to stop immediately upon pressing 'Ctrl c', but it can be brutally effective.  
The option for client renegotiation is given;  if the target server supports client initiated renegotiation, this option should be chosen.
Even if the target server does not support client renegotiation (for example CVE-2011-1473), it is still possible to impact/DOS the server with this attack.  
It is very useful to run this against loadbalancers/proxies/SSL-enabled servers (not just HTTPS!) to see how they cope under the strain.


*Slowloris - uses netcat to slowly send HTTP Headers to the target host:port with the intention of starving it of resources.  This is effective against many, although not all, HTTP servers, provided the connections can be held open for long enough.  Therefore this attack is only effective if the server does not limit the time available to send a complete HTTP request.
Some implementations of this attack use clearly identifiable headers which is not the case here.  The number of connections to open to the target is configurable. 
The interval between sending each header line is configurable, with the default being a random value between 5 and 15 seconds. The idea is to send headers slowly, but not so slow that the servers idle timeout closes the connection.
The option to use SSL (SSL/TLS) is given, which requires stunnel.

Defences against this attack include (but are not limited to):

Limiting the number of TCP connections per client; this will prevent a single machine from making the server unavailable, but is not effective if say, 10,000 clients launch the attack simultaneously.  Additionally, such a defensive measure may negatively impact multiple (legitimate) clients operating behind a forward proxy server.

Limiting the time available to send a complete HTTP request; this is effective since the attack relies on slowly sending headers to the server (the server should await all headers from the client before responding).  If the server limits the time for receiving all headers of a request to 10 seconds (for example) it will severely limit the effectiveness of the attack.  It is possible that such a measure will prevent legitimate clients over slow/lossy connections from accessing the site.


#Disclaimer

This script is only for responsible, authorised use. You are responsible for your own actions and this script is provided without warranty or guarantee of any kind.  The author(s) accept no responsibility or liability on your behalf.


#Also see
Pentmenu is available as a package (https://archstrike.org/packages/pentmenu) on Arch Linux.  Big love to Archstrike (https://archstrike.org/)
