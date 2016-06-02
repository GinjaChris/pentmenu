# pentmenu
A simple bash script inspired by pentbox.

Designed to be a simple way to implement various basic pentesting network functions using, where possible, readily available software commonly installed on most linux distributions, without having to resort to multiple specialist tools.

Currently implemented functions:

*Show external IP (uses curl)

*TCP Port scanner (using netcat)

*Simple banner grabber (using netcat)

*TCP Syn Flood (using hping3 where available, otherwise reverts to nping)

*UDP Flood (using hping3 if available, otherwise reverts to nping)

*SSL DOS (using openssl)

*Slowloris (using netcat)

*Create a TCP or UDP listener (uses netcat)

*More to come...


Sudo is implemented where necesssary.

Tested on Debian and Arch.

Requirements:

sudo 

curl

netcat

hping3 (or nping)

openssl
