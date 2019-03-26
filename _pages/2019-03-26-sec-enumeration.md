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

> Don't like to read stuff? Here's a summary of takeaways from the tutorial below.
> $ masscan <IP/range> -p0-65535,U:0-65535 -e eth0 -oG output.txt | This command will scan an IP or an IP range for all TCP and UDP ports over the eth0 adapter and output a greppable result file to output.txt
> $ nmap -v -p- -sC -sV -oA output.txt <IP> | This command will scan an IP for all TCP ports, print a verbose output, use default nmap scripts for system and version enumeration and output all formats to a file named output.txt

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

-Pn: Treat all hosts as online -- skip host discovery
-sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
-sU: UDP Scan
-p : Only scan specified ports
--top-ports : Scan  most common ports
-sV: Probe open ports to determine service/version info
-sC: equivalent to --script=default
-O: Enable OS detection
--max-retries : Caps number of port scan probe retransmissions.
-T<0-5>: Set timing template (higher is faster)
-v: Increase verbosity level (use -vv or more for greater effect)
-A: Enable OS detection, version detection, script scanning, and traceroute
