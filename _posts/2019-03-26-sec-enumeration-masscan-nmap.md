---
layout: single
classes: wide
title:  "Security Essentials - Enumeration"
date: 2019-03-26
categories:
    - "security essentials"
tags:
    - enumeration
    - boot2root
    - ctf
    - hacking
    - howto
---

Enumeration of any system, host or application is very important before you try your tools and run amok on it. In this post I will talk about two very important tools that security guys use (both bad and good) to get information on a system.

1.  Masscan
2.  Nmap

> Don't like to read stuff?

>Here's a summary of takeaways from the tutorial below.

> $ masscan <IP/range> -p0-65535,U:0-65535 -e eth0 -oG output.txt

> This command will scan an IP or an IP range for all TCP and UDP ports over the eth0 adapter and output a greppable result file to output.txt

> $ nmap -v -p- -sC -sV -oA output.txt <IP>

> This command will scan an IP for all TCP ports, print a verbose output, use default nmap scripts for system and version enumeration and output all formats to a file named output.txt

I highly recommend reading through this post to know more about the tools rather than blindly using commands created by other people.

# Masscan

It is generally used for fast scanning of a network. I will try to include everything there is to know about `masscan`, if you need more information, keep reading my posts on [walkthroughs on hackthebox machines](https://swapnil-pathak.github.io/tags/#machines). I use both `masscan` and `nmap` to enumerate the system.

## Installation

To install `masscan` on your Linux box, you just need to run the following commands on your terminal.

```bash
$ sudo apt-get install clang git gcc make libpcap-dev
$ git clone https://github.com/robertdavidgraham/masscan
$ cd masscan
$ make
```

If you are on a MacOS box, one command is sufficient.

```bash
$ brew install masscan
```

## Man page

General syntax for using masscan:
> $ masscan <ip/range> <options>

### Options that matter

> -p<port/s>, --ports<ports>

This option is used to specify port numbers or a range for both TCP and UDP ports to scan. For example, `masscan x.x.x.x -p0-65535,U:0-65535` can be used to scan ports in range `0-65535`, the `U:` means UDP port range.
This option comes in very handy to identify any open ports in a system so that you could carry out a more targeted enumeration on ports you find open.

> ‐‐top-ports <number>

One of the `nmap` options available in masscan. Can be used like this: `masscan x.x.x.x --top-ports 100`
According to this example, top 100 ports will be scanned for the specified IP.

> --rate <packets-per-second>

This option comes in handy if you need a quick scan of a system. For example, `masscan x.x.x.x -p80,443,8080 --rate=100000` will almost instantly scan the IP for ports 80, 443 and 8080 (Basically, web ports).

> --echo

If you like to configure everything by yourself, this option is for you. Usage: `masscan x.x.x.x --top-ports 100 --rate=100000 --echo > config.txt`
It will create a file named `config.txt` with the IP range and the specified options. This specific configuration can be used later using an option that we will discuss below.

> -c <filename>, --conf <filename>

After creating the `config.txt` file as shown above, you can use that configuration to start a scan like this `masscan -c config.txt`.
Following is a sample configuration file.

```bash
# targets
range = 10.0.0.0/8,192.168.0.0/24
ports = 21,23,80,8080,U:53,161
ping = true
# adapter
adapter = eth0
adapter-ip = 192.168.0.1
```

> --adapter-port <port>

If a port number is specified, packets from the scanning system will be sent from the port as source. An example can be, `masscan x.x.x.x -p80 --adapter-port 60000`
Be sure to filter the port on the host firewall to prevent the host network stack from interfering.

> --adapter-ip <ip>

Similar to the port option, this option can be used to spoof the source IP address to an IP in the range of the local network. Sample usage: `masscan x.x.x.x -p0-65535 --adapter-ip 192.168.0.10`

> -e <ifname>, --adapter <ifname>

In order to specify the network interface that you wish to run the scan from. For example, `masscan x.x.x.x -e eth0`
It will use `eth0` as the preferred interface to run a scan.

# Nmap

There are a lot of tutorials available online trying to explain the use of nmap. The truth is a single post cannot encompass the mighty network mapper. However, most of the time you don't even need to use the full extent of `nmap` abilities.

## Installation

If you are working on a *Kali* box you should have it pre-installed. Otherwise, please refer this [link](https://nmap.org/download.html) for the downloading and installing nmap.

## Man page

General syntax for using nmap:
> $ nmap <options> <IP>

### Options that matter

> -v: Increase verbosity level (use -vv or more for greater effect)

If you would like to know if nmap is working and not just stuck, this option is for you. It shows everything that's going on and presents a detailed result of the scan.

> -Pn: Treat all hosts as online -- skip host discovery

If a packet filtering firewall or a software exists on the system you are scanning, this option can be used. It assumes that the host is live and just try to scan the ports.
Usage: `nmap -v -Pn <ip>`

> -sS/sT: TCP SYN/Connect()

The `-sS` option is used to send SYN packet to all the ports of the IP address. If the server responds with a SYN/ACK packet, it means a port is open. Whereas, in case of the `-sT` option, it attempts to establish a connection to the server after locating an open port and then, resets the connection. (If you want, you can read more about the 3-way handshake for TCP connections [here](https://www.geeksforgeeks.org/computer-network-tcp-3-way-handshake-process/).)
Usage: `nmap -v -sS <ip>` or `nmap -v -sT <ip>`

> -sU: UDP Scan

Some services run on UDP ports such as, DNS, SNMP or DHCP. The `-sU` option is used to scan a server for open UDP ports. Services running on UDP ports are generally easy to break into hence, this option is very important.
Usage: `nmap -v -sU <ip>`

> -p : Only scan specified ports

This is a very nice option to have if you have a targeted attack in mind. You can specify port numbers or simply use `-p-` to scan all the ports (0-65535).
Usage: `nmap -v -p- -sT <ip>` or `nmap -v -p 80,443,8080 <ip>`

> -sV: Probe open ports to determine service/version info

In order to determine the version of the service running on a server, after detection of an open port, the `-sV` option can be used.
Usage: `nmap -v -Pn -sV <ip>`

> -sC: equivalent to --script=default

This option is one of the most important one's in the nmap suite. It uses default NSE (Nmap Scripting Engine) scripts to produce valuable information about the open ports on a server and about the server itself. Tests will be performed on the server with this option to identify any potential vulnerabilities that might be present on the system.
Usage: `nmap -v -Pn -sC <ip>`

> --max-retries <number>: Caps number of port scan probe retransmissions.

When ports are filtered, nmap tries to retransmit packets when they get lost on the network or a poor network reliability exists. The `--max-retries <number>` option, can be used to set the retransmission cap to particular number of packets.
Usage: `nmap -v -p- --max-retries 0 <ip>`

> -T<0-5>: Set timing template (higher is faster)

This option is used to determine the type of scan that you want to perform on the server. After the `-T` option, you specify a number ranging between 0-5 to set the tempo of the scan. (0 is slowest and 5 is fastest)
Usage: `nmap -v -p- -T5 <ip>`

> -oA <basename>: Output in the three major formats at once

In order to output and store scan results in normal, XML, and grepable formats at once, use the `-oA` option. You need to specify the output filename after this option.
Usage: `nmap -v -p- -sV -oA nmap <ip>`

There are plenty of resources online that you could read to become experts in masscan and nmap but trust me, reading gets you nowhere unless you try it own your own.

Happy Hacking! Cheers!
