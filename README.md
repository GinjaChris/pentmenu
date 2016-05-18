# pentmenu
A simple bash script inspired by pentbox.

Designed to be a simple way to implement various basic pentest network functions without having to remember the options for multiple tools.

Currently implemented functions:

*Show external IP (uses curl)

*TCP Port scanner (using netcat)

*Simple banner grabber (using netcat)

*TCP Syn Flood (using hping3 otherwise reverts to nping)

*SSL DOS (using openssl)

*Create a TCP or UDP listener (uses netcat)

*More to come...


The script already implements sudo where required.  If your user does not have sudo capability, run as root.  Tested on Debian and Arch.

Requires:

curl

netcat

hping3 (or nping)

openssl
