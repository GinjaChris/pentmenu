# pentmenu

**一个能让你进行快速简便网络扫描与DOS攻击的命令菜单**


必要时需要`sudo`
已在`Debian`与`Arch`下测试通过

## 依赖：

* bash

* sudo

* curl

* netcat（必须支持'-k'选项，推荐openbsd版本）

* hping3（或者用nping来代替进行洪水攻击）

* openssl

* stunnel

* nmap

* whois（推荐但不要求）

* nslookup（或'host'）

* ike-scan

## 如何使用？


- 下载脚本：

```
$ wget https://raw.githubusercontent.com/GinjaChris/pentmenu/master/pentmenu
```

- 给予运行权限：

```
$ chmod +x ./pentmenu
```

- 运行：

```
$ ./pentmenu
```

也可以在`https://github.com/GinjaChris/pentmenu/releases`下载最新的发布，解压并运行脚本。
或者通过git clone:

```
git clone https://github.com/GinjaChris/pentmenu
```

## 模块相关细节

**RECON MODULES**

* Show IP - 使用`curl`来获取您的外网IP。使用`ip a`或`ifconfig`（如果适宜）显示您的本地IP。


* DNS Recon - 被动侦查，进行一次DNS查询（基于输入目标的正向或反向查询）并进行一次whois查询。如果whois不可用则使用`ipinfo.io`查询（仅对IP查询有效，域名无效）。


* Ping Sweep - 使用`nmap`对目标或网域进行一次 ICMP echo (ping) 操作。


* Quick Scan - TCP端口扫描器使用`nmap`通过`TCP SYN`扫描开放端口。Nmap不会在进行TCP扫描前进行ping扫描。这个模块会扫描最常见的1000个端口。这个模块当然可以对单一目标或整个网络进行扫描，但它的确是被设计用于在IP段内识别存活的主机。扫描将会花费很长时间完成，请耐心等待。


* Detailed Scan - 使用`nmap`来进行识别存活主机、扫描开放端口、尝试识别系统、探测运行的软件版本。Nmap不会在这部分扫描前进行ping扫描。在此模式下，Nmap默认的`User-Agent`被替换为微软Edge浏览器，以此来避免用过HTTP探测。*所有*的TCP端口（无论对于域名、IP或子网）都将被扫描。同时模块可以扫描一个单一主机或许多主机，它的设计目的是对一个单一系统进行信息获取。扫描将会花费很长时间完成，请耐心等待。


* UDPscan - 使用`nmap`扫描开放的UDP端口。*所有*的UDP端口都会被扫描。


* Check Server Uptime - 通过使用`hping3`查询开放的TCP端口来估计目标的存活时间。不同机器查询的结果精确度各不相同。并不能对所有的服务器适用。


* IPsec Scan - 尝试使用ike-scan和各种第1阶段建议来确定IPsec VPN服务器的存在。此模块任何文字输出，无论被认为是“握手”还是“没有可匹配的提议”，都指向了IPsec VPN服务器的存在。浏览 http://nta-monitor.com/wiki/index.php/Ike-scan_User_Guide 来了解ike-scan和 VPN phase 1.


**DOS MODULES**

* ICMP Echo Flood - 通过`hping3`发送传统ICMP Echo flood攻击目标。对于现代系统可能收获不大，但对于通过攻击防火墙来测试他们反应来说很有效。`Ctrl+C`停止攻击。
洪水攻击包的来源地址是可以设置的。注意目标可以是一个IP地址（如127.0.0.1）或一个域名（如localhost.localnet.com）。目标*不要*包含协议（如http://localhost.localnet.com ）。


* ICMP Blcknurse Flood - 使用`hping3`发送ICMP flood攻击目标。这些ICMP 包是“Destination Unreachable, Port Unreachable”类型的。这种攻击可能会对某些系统造成CPU大量占用的情况。使用`Ctrl+C`停止攻击。浏览 http://blacknurse.dk/  获取更多相关信息。洪水攻击包的来源地址是可以设置的。


* TCP SYN Flood - 通过`hping3`发送TCP SYN洪水攻击。如果没有找到`hping3`，它会尝试使用 `nmap-nping`来代替。Hping3因为其尽可能多的发包而被推荐。您可以在选项中使用您自己的IP发包、使用指定的某个IP（比如用来恶作剧）或对每一个包使用随机源IP（滑稽~）。
作为可选功能，您可以向每个SYN包加入数据。所有的SYN包设置了分片并使用hping的16字节虚拟MTU，用于保证包的分片。
使用`nmap-nping`意味着将会每秒发送X个包直到Y个包已被发送，并且只支持使用自己的IP或指定的IP。
TCP SYN 洪水不太可能使服务器宕机，但用来测试交换机、防火墙设备稳定性会非常不错。
请注意hping会同时报告出口与IP，这有可能会使您以为脚本没有按照预期运行，实际上源IP*会*被正确的设置，如果您存疑请查看抓包的数据。
因为其源IP可以被设置，所以很容易就可进行 LAND 攻击（浏览https://en.wikipedia.org/wiki/LAND 了解相关）。源IP可控的能力同样可以使得发送SYN包去目标并迫使目标向第二个目标发送SYN-ACK包。


* TCP ACK Flood - 提供与SYN flood相同的选项，不过设置的是ACK TCP标志。
有的系统可能会花费过多的CPU资源循环处理这类包。如果源IP被设置为来自某一个已经稳定建立的连接，那个已经稳定的连接可能会被攻击至中断。这种攻击被认为是“盲打”是因为它没有考虑任何已建立连接的任何细节（如顺序或已经确认的数量）。


* TCP RST Flood - 提供与SYN flood相同的选项， 不过设置的是RST TCP 标志。
如果源IP被设置为来自一个已经稳定建立的连接，该连接会被中断。
浏览 https://en.wikipedia.org/wiki/TCP_reset_attack 了解相关。


* TCP XMAS Flood - 与SYN洪水和ACK洪水相似，拥有相同的选项，但发送所有TCP标志（CWR、ECN、URG、ACK、PSH、RST、SYN、FIN）。这种包被认为是“组装得像一棵圣诞树”。至少从理论上来说，接收者需要花费比标准形式包更多的资源来处理这种包。
然而这种包指向了一种不寻常的行为（如攻击）并且会被入侵检测系统轻易地识别。


* UDP Flood - 很像TCP SYN Flood但对于指定目标:端口发送UDP包。与TCP SYN Flood函数相同，使用`hping3`并在没有找到时使用`nmap-nping`作为替代。所有的选项与TCP SYN Flood相同，除了你必须指定UDP包的内容。
再次声明，这对测试交换机或VOIP系统来说是个好的方法。


* SSL DOS - 使用`OpenSSL`来尝试对目标的端口进行拒绝服务攻击。原理是建立许多连接并使服务器进行许多代价昂贵的握手计算操作。这段代码写的并不好看，所以不要期望它在您按下`Ctrl+C`后能够立即停止，但它的效率十分残酷。
脚本提供关于客户端重连的选项，如果目标服务器支持客户端发起的重连，您就应该启用该选项。
就算目标服务器不支持以上特性（如CVE-2011-1473），它仍然有可能影响或使服务器拒绝服务。
用这种方法攻击负载均衡服务器、代理服务器或启用SSL的服务器（不只是HTTPS，只要是任何SSL或TLS加密的服务）来观察它们如何应对压力是非常有效的。


* Slowloris - 使用`netcat`来缓慢地向目标:端口发送HTTP消息头，目的是使其缺乏资源。它可以对付许多（可惜不是所有）HTTP服务器，使得连接可以保持足够长的时间。因此这种攻击只能对没有限制能够完整发送HTTP请求的时间的服务器有足够效果。
有一些对这种攻击的执行使用了可以清楚识别的消息头，本脚本进行这样的操作。此外，与目标进行的连接数是可以设置的。
发送每一行消息头之间的间隔是可以设置的，默认随机在5到15秒之间。思路是缓慢地发送包，但不是太慢以至于服务器超时而终止连接。举个例子，如果我们每900秒发送一行消息头，服务器很有可能远在我们发送下一行之前就终止这个连接。
您可以选择是否使用SSL（SSL或TLS），这将需要使用到`stunnel`并使得攻击HTTPS可行。您不能在攻击明文HTTP服务器时选择使用SSL选项。
对于这种攻击的防御方式包括但不限于：
1. 限制每个客户端可进行的TCP连接。这可以防止单一的一台机器就使服务器不可用，但如果像100000个客户端同时进行攻击的话，这样做就并不效率。并且，这种防御方式可能会对多个客户端（合法地）通过同一个远端代理服务器进行的操作产生消极的影响。
2. 限制能够完整发送HTTP请求的时间，这很有效，对于攻击依赖于向服务器缓慢地发送消息头（服务器要在响应前接收所有的消息头）。如果服务器限制了最大接收所有消息头的时间为10秒（为例），那将会严重限制这种攻击能够产生的影响。这种操作可能会阻止合法的客户端通过缓慢或丢包严重的连接来访问网站。


* IPsec DOS - 使用`ike-scan`来尝试对指定的IP进行洪水攻击，可使用随机源IP的主模式和积极模式第一步包。使用IPsec Scan模组来确定IPsec VPN服务器的存在。


* Distraction Scan - 这实际上并不是一种DOS攻击，它只是用`hping3`进行许多的TCP SYN扫描（用您恶意指定的源IP，比如您最大的敌人）。它被设计来进行很明显的扫描来引起入侵检测系统的注意并期望掩盖其他实际扫描以及您的其他操作。


* DNS NXDOMAIN Flood - 这种攻击使用`dig`并且被设计用来发送大量对不存在域名的查询压测您的DNS服务器。当对递归查询的DNS服务器使用时，它会增加服务器负载并使缓存中充满无用的响应，使得对合法请求的响应变慢或终止。攻击最好在多个客户端上进行。使用`Ctrl+C`来停止攻击。


**EXTRACTION MODULES**

* Send File - 这个模块使用`netcat`发送TCP或UDP数据。对于提取数据十分有用。
Md5和sha512校验码在发送文件前会被计算并显示。
文件可以被发送至您想要的服务器，Listener被用来接收这些文件。


* Listener - 使用`netcat`在指定的TCP或UDP端口打开一个监听器。对于测试系统日志的连接度、接收文件或探测网络中的扫描很有用。所有接收到的数据会被写入到`./pentmenu.listener.out`或您指定的文件。
如果您在UDP端口接收文件，监听器必须手动`Ctrl+C`关闭。因为我们必须强制令netcat保持开放（UDP是无连接协议）。
如果您在TCP端口接收文件，连接会在客户端终止发送（文件传输完成）后自动关闭，并且md5和sha512校验码会自动计算并显示。


## 免责声明

这个脚本仅能被授权并负责地使用。您必须对您的行为负责，脚本不提供任何承诺或保障。作者（们）对您的行为没有任何责任和法律义务。


## 另外

Pentmenu 目前在 Arch Linux 中作为一个可用的[软件包](https://archstrike.org/packages/pentmenu)。向[ArchStrike](https://archstrike.org/) 和[Parrot linux](https://www.parrotsec.org/) 表明我的热爱。


## 打赏

您可以通过加密货币打赏：


Bitcoin:
```
bc1q96r5gr35qeah3q8asme63zhusjrfx6vg9jnaf5
```
