#pentmenu


A simple bash script inspired by pentbox.

Designed to be a simple way to implement various basic pentesting network functions using, where possible, readily available software commonly installed on most linux distributions, without having to resort to multiple specialist tools.

Currently implemented functions:

*Show external IP

*TCP Port scanner

*Simple banner grabber

*TCP Syn Flood

*UDP Flood

*SSL DOS

*Slowloris

*Create a TCP or UDP listener

*More to come...


Sudo is implemented where necesssary.

Tested on Debian and Arch.

Requirements:

sudo 

curl

netcat

hping3 (or nping)

openssl


#How to use?


- Download the script:

$wget https://raw.githubusercontent.com/GinjaChris/pentmenu/master/pentmenu

- Make it executable:

$chmod +x ./pentmenu

- Run it:

$./pentmenu
